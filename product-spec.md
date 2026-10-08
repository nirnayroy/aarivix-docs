# Product Specification

> Protected — change only via ADR.

## What Aarivix is

Aarivix is a drone-photogrammetry SaaS / construction-intelligence platform. Users upload drone imagery; the backend orchestrates 3D reconstruction — orthomosaics, point clouds, DEMs, meshes, and reports — via photogrammetry engines (NodeODM default, WebODM optional).

## Core user journey (invariant)

1. **Create a project** — the top-level container for a site.
2. **Create a capture session** under the project — one flight / acquisition.
3. **Upload imagery** into the session — source media is persisted to S3 with checksums.
4. **Trigger reconstruction** — a photogrammetry job runs the source images through the provider.
5. **Receive artifacts** — outputs are stored in S3 and registered as records; downloads are served via presigned URLs.

This ordering is a product invariant: every artifact is traceable back through job → session → project.

## Non-negotiable invariants

- **Source imagery is never mutated.** The source bucket is read-only to the pipeline; outputs go to a separate artifact bucket.
- **Every output is traceable** to the job and source session that produced it.
- **Jobs are observable** — a job always has a status (PENDING / RUNNING / SUCCESS / FAILED / CANCELED) and is never silently lost.
- **Artifacts are addressable** via stable S3 paths (`outputs/jobs/{job_id}/...`) and time-boxed presigned URLs, not public links.

## Out of scope (today)

- Multi-tenancy — currently single-tenant (hardcoded default org).
- Authentication — no auth layer wired up yet.
- Real-time collaboration / annotations / measurements / reports — model stubs exist; not built.
