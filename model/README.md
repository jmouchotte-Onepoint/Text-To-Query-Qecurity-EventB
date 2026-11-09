# Rodin project

Event-B model for *Verify the Query, Not the Generator*. See the
[root README](../README.md) for the refinement chain, proof status, and
citation.

The `.project` descriptor names this project **`v6.0`** — that is the name
Rodin will show in the Event-B Explorer, regardless of the directory name.

## Import into Rodin

Rodin 3.x: `File > Import > General > Existing Projects into Workspace`, then
select this `model/` directory. The builder regenerates the derived files and
reattaches the committed proofs to the regenerated obligations.

## File types

| Extension | Role | Versioned |
|-----------|------|:---------:|
| `.bum` | machine (source) | ✓ |
| `.buc` | context (source) | ✓ |
| `.bpr` | proof trees — manual work, not regenerable | ✓ |
| `.bps` | proof status (confidence per obligation) | ✓ |
| `.project` | Rodin project descriptor | ✓ |
| `.bcm` | statically-checked machine, derived from `.bum` | — |
| `.bcc` | statically-checked context, derived from `.buc` | — |
| `.bpo` | generated proof obligations | — |

Derived files are listed in the root `.gitignore`: committing them would add
~3 MB of churn per rebuild for no benefit, since the Rodin builder reconstructs
them deterministically from the sources.
