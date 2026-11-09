# Verify the Query, Not the Generator

**Engine-Agnostic Event-B Proofs for Text-to-Query Security with Data-Centric Access Control**

Formal verification artifact accompanying the manuscript submitted to the
*Journal of Computer Languages*.

Jean Mouchotte (Onepoint / Paris Panthéon-Assas University) ·
Lazhar Hamel (Efrei Research Lab) ·
Layth Sliman (Efrei Research Lab)

---

## What this is

Text-to-query systems translate a natural-language intent into an executable
query built dynamically. A single syntactic variation is enough to turn a
legitimate action into an access-policy violation or a leak of sensitive data.
The question this model answers is: **before execution, is a query structurally
consistent, secure, and authorized for this user in this data context?**

This repository holds the Event-B model that decides it, and the machine-checked
proofs that it is correct. Six machines (`M1`–`M6`) introduce the requirements
progressively — from the operation type to the decision to execute or block —
without weakening any property established at a previous level. Authorization is
derived at each execution from data metadata, user attributes, and execution
context.

The model is **agnostic to the generator**. A proof tied to one LLM would age
with that model; these models keep evolving and remain fallible. So the proof
targets the query, not the thing that wrote it.

## Refinement chain

| Machine | Introduces | Rejects |
|---------|-----------|---------|
| `M1_Query` | operation type, raw segments | unsupported query type, dangerous segment |
| `M2_Database` | namespaces, entities, attributes | element outside the allowed schema, unsafe structure |
| `M3_Conditions` | predicates (column, operator, value) | forbidden operator, dangerous condition fragment |
| `M4_Links` | joins / lookups between entities | unsafe link, forbidden link type or operator |
| `M5_Authorizations` | authorization scope, user & data metadata | out-of-scope access |
| `M6_Response` | selected data, execution decision | data not allowed, not found, or forbidden |

```
M0_Init_Context
  └─ M1_Query_Context ─────────── M1_Query
       └─ M2_Database_Context ─── M2_Database
            └─ M3_Conditions_Context ─ M3_Conditions
                 └─ M4_Links_Context ── M4_Links
                      └─ M5_Authorizations_Context ─ M5_Authorizations
                                                      └─ M6_Response
                                                           └─ Machine_Test
```

## Engine instantiation

The generic model is instantiated on a reference HR dataset for one relational
and two non-relational engines. The same queries yield the same decisions on
every engine — only the profile contexts change, because the notions shared by
all engines form the generic model.

| Policy context | Accept | Reject (authorization) | Reject (engine-specific) |
|----------------|--------|------------------------|--------------------------|
| `SQL_HR_Policy_Context` | ✓ | ✓ | SQL injection |
| `Mongo_HR_Policy_Context` | ✓ | ✓ | regex denial-of-service |
| `Cassandra_HR_Policy_Context` | ✓ | ✓ | `ALLOW FILTERING` |

## Proof status

All proof obligations are discharged automatically by Rodin — **137 / 137**, every
one at confidence 1000 (`DISCHARGED`). Counts below are read from the committed
`.bps` status files.

| Component | POs | Discharged |
|-----------|----:|-----------:|
| `M1_Query` | 4 | 4 |
| `M2_Database` | 9 | 9 |
| `M3_Conditions` | 16 | 16 |
| `M4_Links` | 19 | 19 |
| `M5_Authorizations` | 20 | 20 |
| `M5_Authorizations_Context` | 4 | 4 |
| `M6_Response` | 29 | 29 |
| `SQL_HR_Policy_Context` | 4 | 4 |
| `SQL_HR_Query_Accept` | 3 | 3 |
| `SQL_HR_Query_RejectAuth` | 3 | 3 |
| `Mongo_HR_Policy_Context` | 4 | 4 |
| `Mongo_HR_Query_Accept` | 3 | 3 |
| `Mongo_HR_Query_RejectAuth` | 3 | 3 |
| `Mongo_HR_Query_RejectRegexDos` | 6 | 6 |
| `Cassandra_HR_Policy_Context` | 4 | 4 |
| `Cassandra_HR_Query_Accept` | 3 | 3 |
| `Cassandra_HR_Query_RejectAuth` | 3 | 3 |
| **Total** | **137** | **137** |

Components not listed (`M0_Init_Context`, the intermediate `*_Context` files,
`Machine_Test`, `SQL_HR_Query_RejectInjection`,
`Cassandra_HR_Query_RejectAllowFiltering`) generate no proof obligation: they
carry hypothesis sets only.

## Reproducing the proofs

Requires [Rodin](https://www.rodinp.org/) 3.x.

1. `git clone https://github.com/jmouchotte-Onepoint/Text-To-Query-Qecurity-EventB.git`
2. In Rodin: `File > Import > General > Existing Projects into Workspace`,
   select the `model/` directory.
3. The Rodin builder regenerates the static-check files (`.bcm`, `.bcc`) and the
   proof obligations (`.bpo`), then reattaches the committed proofs (`.bpr`) to
   the regenerated obligations.
4. Open the *Event-B Explorer* — every component should show a green
   (fully discharged) marker. To re-prove from scratch instead, delete the
   `.bpr`/`.bps` files and run `Project > Clean`, then the auto-prover.

## Repository layout

```
model/            Rodin project (Event-B sources + saved proofs)
  *.bum           machines  (source)
  *.buc           contexts  (source)
  *.bpr  *.bps    proof trees and their status
  .project        Rodin project descriptor
  README.md       import notes and file-type reference
.gitattributes    diff/linguist handling for Rodin XML
CITATION.cff      citation metadata
```

Derived files (`.bcm`, `.bcc`, `.bpo`) are not versioned — see `.gitignore`.
They are reconstructed by the Rodin builder on import.

## Citation

See [`CITATION.cff`](CITATION.cff), or use GitHub's *Cite this repository*
button. Please cite the manuscript rather than the repository where possible.

## Contact

Jean Mouchotte — <j.mouchotte@groupeonepoint.com> (corresponding author)
