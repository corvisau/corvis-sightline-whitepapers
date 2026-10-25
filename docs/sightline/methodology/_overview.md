The method models the security architecture of a system with security zones, and it aligns in part with IEC 62443. The methodology pages explain what each part of a model means and when to choose it. The usage pages explain how to author and render a model.

## Pages

| Page | Question it answers |
|---|---|
| [Tetra entities](/docs/sightline/methodology/tetra-entities) | Which entity represents a boundary, a device, a network, a piece of information or a collection of systems? |
| [Tetra channels and flows](/docs/sightline/methodology/tetra-channels-and-flows) | When is a connection a channel, when is it a flow, and which type fits? |

Methodology pages for Lamina and Metron are not written yet.

## The method in short

1. Model the security architecture, not the physical architecture. A zone groups nodes that share common security requirements and need a common boundary. A zone can follow an existing network security control, such as a firewall, but it does not have to.
2. Draw relationships between the nodes. A channel states that information could move between two nodes, and a flow states what information moves.
3. Keep one model for one system. Use [data slices](/docs/sightline/usage/tetra/data-slice-filtering) for different views of the same model rather than separate copies.
4. Let rules read the structure. A zone boundary that a channel or flow crosses is a fact the model holds, and a rule reports it as a [record](/docs/sightline/usage/tetra/rules-and-findings).
