A Tetra network records the address ranges it represents in its **Addressing**
list. The list replaces addressing written into the free-text description, where
it cannot be queried, joined against observed network data or checked for
mistakes.

**Addressing** is optional. One network can hold several ranges. A network in a
security model abstracts the as-built network, and often stands for more than one
real range.

---

## Setting addressing in the UI

Where you edit a network decides which editor you get.

**Creating a network.** The **New network** drawer takes a list of ranges. Type
`172.21.100.0/24`, press Enter, and repeat for each range. This surface offers no
notes or known instances, because the network does not exist yet.

**Editing an existing network.** Select the network to open the props drawer and
use the **Addressing** list:

- **+ Add** appends a new empty entry.
- **Edit →** on a row opens a form with **Range**, **Note** and **Known instances**.
  Type **Known instances** as comma-separated values, for example `235, 184, 206`.
- **×** removes a row.

The Tabular Editor has a read-only **addressing** column that shows each network's
ranges, comma-joined. It is off by default; enable it from the Column Manager.

## Templated ranges

A part of a range in braces, such as `{site}`, stands for any single octet from 0
to 255. Type it in the **Range** field. It lets one network represent the same
network at every site. Two stations are then one network, and a flow between
them is a flow between two templated networks.

The range `10.20.{site}.0/24` describes a class of network. Without templating, a
station network needs a near-identical network per site or a `/16` range that
matches every site at once.

Rules:

- The mask governs matching and the placeholder constrains nothing. A placeholder
  marks which octet varies and binds to whatever a candidate has there. It is
  legal in any octet, whether or not the mask covers it.
- `10.30.{site}.0/16` means anything inside `10.30.0.0/16`, with the third octet
  as the per-site octet. It matches `10.30.235.0/26`, `10.30.12.128/25` and
  `10.30.7.9/32`. It does not match `10.31.1.0/24` or the broader `10.0.0.0/8`.
- Matching `10.30.235.0/26` reports `site=235`, so one network gives a per-site
  view.
- A range can hold more than one placeholder, such as `10.{function}.{site}.0/24`.
  Each placeholder binds.
- A placeholder name can appear once in a range.
- A name belongs to its range. Two ranges that both use `{site}` do not assert the
  same site.
- Two networks can share a template.

### Known instances

**Known instances** records the values you happen to know for the placeholders.
Tetra treats the list as incomplete and reports an unlisted value as new, not as a
mismatch. A range with no known instances accepts any value.

Use **Known instances** on a range with one placeholder. A list of known instances
on a range with no placeholder is an error. So is a plain list on a range with two
or more placeholders.

---

## Problems with a range

Tetra reports an addressing problem as a model diagnostic that names the network
and the entry. A malformed entry never removes the network from the diagram.

Errors:

- A range that is not a valid IPv4 range in `a.b.c.d/prefix` form.
- An empty **Range**.
- Host bits set. Every bit below the prefix split must be zero, so `172.20.23.5/24`
  and `10.1.0.5/25` are errors, while `10.1.0.128/25` and `10.1.0.64/26` are valid.
- A placeholder name repeated within one range.
- A known instance outside 0 to 255 or one that is not a whole number.
- An IPv6 range. IPv6 is not supported.

Warning:

- Two different networks declare overlapping concrete ranges. This is legal in a
  security model but usually worth a second look. Templated ranges are never
  compared this way, and neither are two ranges on the same network.

---

## Querying from a data slice

`addressing` is reachable from the data slice DSL the same way `tags` is.

Networks with nothing declared (a completeness view over the model):

```
model |> where("item.kind == 'network' && item.addressing |> isEmpty()") |> add()
```

Networks declaring a particular range:

```
model |> where("item.addressing |> covers('172.20.23.0/24')") |> add()
```

`covers()` matches on the address rather than on string equality. The
[Address-aware helpers](#address-aware-helpers) section below describes it and
its two companions.

### Raw entries

`item.addressing` holds each entry as it was entered. A range entered without a
note can appear as a plain string, and a range with a note or known instances
appears as an object. A lambda over the list sees a plain string as a string and
an object as an object. A `.range` lambda therefore matches object entries only,
because a plain string has no `.range` property:

```
model |> where("item.addressing |> anyOf(.range == '172.20.23.0/24')") |> add()
```

Use `covers()` for any query that must see both forms.

### Address-aware helpers

Three transforms read addresses rather than strings. Unlike `.range` above, they
read both entry forms.

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
> not fail the rest of the call. A bad value annotates the model, it does not
> delete a node. A misspelled candidate therefore looks exactly like no match.
> Check the model diagnostics if a query returns less than you expect.

---

## Related

- [Tetra Data Slice Filtering](/docs/sightline/usage/tetra/data-slice-filtering): the filter language that queries
  `item.addressing`.
