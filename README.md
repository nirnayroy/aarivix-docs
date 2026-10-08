# Aarivix Documentation

Living documentation for the Aarivix drone-photogrammetry platform. Claude maintains this tree as the codebase evolves — design before building, record how things shipped, log decisions.

## Map

| Folder | What goes here | What does **not** |
|--------|----------------|-------------------|
| [`reference/`](reference/) | **Protected** core spec, principles, contracts that must not drift. | Anything that changes per-feature or per-sprint. |
| [`architecture/`](architecture/) | How the system is structured today (system / backend / frontend). | Proposals not yet built (use `design/`). |
| [`design/`](design/) | Proposals, RFCs, component / API / UX design **before** it's built. | Records of decisions already made (use `decisions/`). |
| [`implementation/`](implementation/) | How shipped features actually work + notable change notes. | Speculative design (use `design/`). |
| [`decisions/`](decisions/) | ADRs — one file per architectural decision. | General docs or how-tos. |
| [`operations/`](operations/) | Runbooks, deploy / rollback, incident response, infra ops. | Architecture rationale (use `architecture/`). |
| [`testing/`](testing/) | Test strategy + how to run unit and pipeline tests. | Test result snapshots from one run (those live with the run). |
| [`api/`](api/) | Endpoint reference + request/response contracts. | Internal service interfaces (use `architecture/`). |
| [`setup/`](setup/) | Local / Docker / environment setup guides. | Production ops (use `operations/`). |

## Governance

- **`docs/reference/` is protected.** Its files encode invariants that should survive refactors. Change them **only** via an ADR in [`decisions/`](decisions/) — never silently.
- Keep docs current as code changes: new design → `design/`, shipped behavior → `implementation/`, a choice with trade-offs → `decisions/`.
- The two WAT engineering principles (specific workflows + explicit failure; single/deterministic/loud tools with usage examples) live in [`reference/principles.md`](reference/principles.md) and are binding.
- The `/sc:` skill → WAT-layer mapping lives in [`reference/wat-toolkit.md`](reference/wat-toolkit.md).

## Legacy / historical notes (repo root)

Pre-existing knowledge still lives loose in the repo root and is kept for history — distil into the folders above when revisited, don't treat as current truth:

- `PHOTOGRAMMETRY_SESSION_CHANGES.md`, `PHOTOGRAMMETRY_WORKFLOW.md` → source for `implementation/`.
- `PHOTOGRAMMETRY_TEST_RESULTS.md` → source for `testing/`.
- `chore_BE_scaffold.md` → backend scaffold notes.
