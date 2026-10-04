A Tetra network object can record the address ranges it represents in an optional
`addressing:` list. The list replaces addressing written into the free-text
`description`, where it cannot be queried, joined against observed network data,
or checked for mistakes.

`addressing` is optional and additive.

---

## The shape

```yaml
networks:
  # 1. concrete single range
  - id: net-core-a
    label: Core Network A
    addressing:
      - 172.20.23.0/24

  # 2. several ranges on one security-model object
  - id: net-shared-services
    label: Shared Services Network
    addressing:
      - range: 172.20.20.0/24
      - range: 172.22.30.0/24
        note: physical pair at the second site

  # 3. templated range - one per site, without enumerating sites
  - id: net-station-a
    label: Station A Network
    addressing:
      - range: 10.20.{site}.0/24
        note: field devices, VLAN 100
        instances: [235, 184, 206]
```

Each entry is either a bare CIDR string or an object. The bare string is
shorthand for `{ range: <string> }` and exists so the common single-range case
stays terse. The `device.connections:` list uses the same two-form convention.

| Field | Type | Notes |
|---|---|---|
| `range` | `string` | Required in the object form. `a.b.c.d/prefix`, where any octet may be a `{placeholder}` |
| `note` | `string?` | Free text about this specific range |
| `instances` | `number[]` or `{ <placeholder>: number[] }`? | Known values for the range's placeholders. Never exhaustive |

**A list, not a scalar, on purpose.** A security-model network deliberately
abstracts the as-built network, so one model object routinely stands for several
real ranges. Forcing one range per object would either explode the object count or
push authors back to writing `/16`s that match everything.

---

## Authoring it in the UI

Where you edit a network decides which editor you get, because the two surfaces
can offer different things.

**Creating a network.** The props drawer's create surface takes a plain
list of CIDRs: type `172.21.100.0/24`, press Enter, repeat. That writes the
bare-string form (case 1 above). Notes and templated instances are not offered
here, because the create surface has no drill-in and the network does not exist
yet.

**Editing an existing network.** Select it and use the **Addressing** list in the
props drawer. Each row drills in to its own form with `range`, `note` and
`instances`, so this is where a bare string becomes an object.

## Templated ranges

`{name}` stands for any single octet, 0–255. It is what lets one object represent
the same network at every site instead of becoming an as-built inventory.

A network may be allocated in two dimensions, for example
`10.<function>.<site>.<host>` across a hundred sites. Without templating, modelling
a station network has three bad options. The author can enumerate a near-identical
object per site, write `10.20.0.0/16` and match every site at once, or invent a
dummy octet that corresponds to nothing. With templating, two stations are one
object, and a flow between two stations is a flow between two templated networks.

Rules:

- **The mask governs matching; the placeholder constrains nothing.** A
  placeholder records which octet varies and binds to whatever a candidate has
  there. It never narrows what matches, and it is legal in any octet, whether or
  not the mask covers it.

  ```
  10.30.{site}.0/16   ==   "anything inside 10.30.0.0/16",
                            noting that the third octet is the per-site octet
  ```

  So it matches `10.30.235.0/26`, `10.30.12.128/25` and `10.30.7.9/32`
  alike, and does not match `10.31.1.0/24` or the broader `10.0.0.0/8`.
  This matters because real per-site allocations are ragged. In one
  network `/26` and `/29` masks are common and vary within one functional octet.
  An author must be able to assert the `/16` without being right about every
  site's mask.
- **The bound value is reported.** Matching `10.30.235.0/26` yields
  `site=235`, so a per-site view is derivable from one model object.
- More than one placeholder is allowed: `10.{function}.{site}.0/24`,
  `10.{fn}.{site}.0/8`. Every one binds.
- **A placeholder name may appear at most once per range.**
- Names are scoped to their range. Two ranges both using `{site}` are not
  asserting the same site.
- A templated range describes a class of network. Two network objects may
  legitimately share a template.

### `instances`

`instances` records the values you happen to know. A consumer always treats the
list as non-exhaustive and reports an unlisted value as new, not as a mismatch.
A range without `instances` accepts any value.

```yaml
      - range: 10.20.{site}.0/24
        instances: [235, 184, 206]          # one placeholder: a bare list

      - range: 10.{function}.{site}.0/24
        instances:                          # two or more: key it by name
          function: [193, 194]
          site: [235, 184]
```

A bare list against a multi-placeholder range is an error, as is `instances` on a
range with no placeholder at all.

---

## Validation

Addressing problems are reported as model diagnostics naming the network id and
the entry index, for example:

```
Network "net-control-prod" addressing[0]: "10.20.0.0/2x" is not a valid IPv4 CIDR
range; expected four octets and a /0-32 prefix, where an octet may be a
{placeholder}.
```

**A malformed entry never removes the network from the diagram.** `addressing`
is an optional annotation, so a typo in one does not delete a node. The trade-off
is that the generated JSON Schema for `addressing` is permissive, so a YAML editor
does not flag mistakes. The app's diagnostics are the feedback path.

Reported as errors:

- A malformed CIDR, or an entry that is not a string or an object.
- A missing, empty, or non-string `range`, including a misspelled `ranges:` key.
- An `addressing` value that is not a list at all (a forgotten YAML `-`).
- **Host bits set.** Checked at bit level, not per octet: every bit below the
  prefix split must be zero. `172.20.23.5/24` and `10.1.0.5/25` are errors;
  `10.1.0.128/25` and `10.1.0.64/26` are valid network addresses.
- A repeated placeholder name within one range. (A placeholder's position is
  not restricted; see the Templated ranges section above.)
- An `instances` value outside 0–255, a non-integer one, a bare list against a
  multi-placeholder range, or a key that is not a placeholder in that range.
- An IPv6 range, which is named specifically rather than failing generically.
  IPv6 is not supported.

Reported as a warning, never an error:

- Two different networks declaring overlapping concrete ranges. Legal in a
  security model, but usually worth a second look. Templated ranges are never
  compared this way, because one legitimately spans many networks. Two ranges on
  the same network are not compared either, since that is what the list is for.

---

## Querying from a data slice

`addressing` is reachable from the data slice DSL the same way `tags` is.

Networks with nothing declared (a completeness view over the model):

```
model |> where("item.kind == 'network' && item.addressing |> isEmpty()") |> add()
```

Networks declaring a particular range, in either entry form:

```
model |> where("item.addressing |> covers('172.20.23.0/24')") |> add()
```

`covers()` matches on the address rather than on string equality. The
[Address-aware helpers](#address-aware-helpers) section below describes it and
its two companions.

### Raw entries

`item.addressing` holds each entry exactly as the author wrote it. A lambda over
the list sees a bare CIDR string as a string and an object entry as an object.
A `.range` lambda therefore matches object entries only, because a bare string
has no `.range` property:

```
model |> where("item.addressing |> anyOf(.range == '172.20.23.0/24')") |> add()
```

Use `covers()` for any query that must see both forms.

### Address-aware helpers

Three transforms read addresses rather than strings. Unlike `.range` above, they
read both entry forms, the bare CIDR string and the object, so the shorthand
is safe to query.

| Helper | Answers |
|---|---|
| `covers(…candidates)` | Does any declared range **contain** this address or subnet? |
| `overlaps(…ranges)` | Does any declared range share space with this range? |
| `isTemplated()` | Does any declared range carry a `{placeholder}`? |

This query finds which network owns an address. The templated range matches, which
is the point of templating:

```
model |> where("item.addressing |> covers('10.20.7.9')") |> add()
```

A bare address means a single host, so the `/32` suffix is not
required. `covers` tests containment, not overlap: a candidate broader than the
declared range does not match. For the symmetric question of whether two ranges
could collide, use `overlaps`:

```
model |> where("item.addressing |> overlaps('10.1.0.0/16')") |> add()
```

Which networks are modelled as a class rather than an instance:

```
model |> where("item.kind == 'network' && item.addressing |> isTemplated()") |> add()
```

> **They never error, they only fail to match.** A missing `addressing`, a value
> that is not a list, an unparseable range, an IPv6 argument or a typo in a
> candidate all yield `false`. Both sides skip independently: a malformed entry
> does not blind the other entries on that network, and a malformed argument does
> not fail the rest of the call. This mirrors the validation design above: a bad
> value annotates the model, it does not delete a node. A misspelled candidate
> therefore looks exactly like no match. Check the diagnostics panel if a query
> returns less than you expect.

---

## Editing in the app

Select a network to open the props drawer. **Addressing** renders as a list:

- **+ Add** appends a new empty entry.
- **Edit →** on a row opens a sub-form with `range`, `note` and `instances`
  (`instances` is typed as comma-separated values, e.g. `235, 184, 206`).
- **×** removes a row.

The Tabular Editor has a read-only **addressing** column showing each network's
ranges comma-joined. It is off by default; enable it from the Column Manager.

**Either authored form edits the same way.** Drilling into a bare-string entry
shows its CIDR in the `range` field, and saving writes the entry back in object
form: `- 10.0.0.0/24` becomes `- range: 10.0.0.0/24` once you edit it. New rows added
from the drawer are always object form.

---

## Related

- [Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering): the filter language that queries
  `item.addressing`.
