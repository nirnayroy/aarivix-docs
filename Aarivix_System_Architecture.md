# Aarivix System Architecture

> Source of truth. **Updated July 5, 2026** for the reduced committed scope (see `Aarivix_MVP_Roadmap.md` v3). Incorporates ADR-001 (`Aarivix_Architecture_Review.md`) and ADR-002 (BIM/viewer stack). The change from the July 4 revision: **the BIM Registration Engine moves from first-class critical component to a planned internal track — in the MVP, timeboxed, not client-committed, undisclosed until working.** The new first-class capability is **multi-epoch spatial consistency** — every capture session of a project lives in one spatial frame, which is what makes cross-session comparison and world-anchored snags work.

---

## Architectural principle

Aarivix **consumes** reconstruction (WebODM/ODM); it does not replace ODM internals. Our engineering is the layer *above* ODM. Priorities under the v3 scope:

1. **Multi-epoch spatial consistency** — all sessions of a project co-registered via GCP/RTK georeferencing in a shared CRS. Cheap to enforce, and everything committed depends on it.
2. **Inspection loop** — measurement (WebODM-grade UX), click-to-image, cross-session compare, world-anchored snag annotation.
3. **BIM Registration Engine (R1–R4)** — a **planned internal track** in the MVP (roadmap Epic O): concierge manual alignment, hard-timeboxed, gated Jul 13. Not client-committed and not disclosed until it works, but built — it is the on-ramp to the post-conversion wedge.

**Core decision (ADR-001, unchanged):** Aarivix runs as **its own Django service with its own schema**, submitting jobs **directly to NodeODM** (`pyodm`). WebODM's webapp is an optional ops UI, not a dependency. The provider contract in `docs/reference/pipeline-contract.md` is protected — nothing in v3 touches it; the per-session pipeline is simply run once per epoch.

---

## Current Development Architecture

### Frontend
- Next.js · React · TypeScript · TailwindCSS

### Backend
- Django · Django REST Framework (separate service from WebODM — ADR-001)

### Async processing
- **Celery + Redis** — own queues/tasks. Existing chain `submit_odm_task → poll_odm_task → download_artifacts_task` untouched; it now runs **per capture session** with no cross-session coupling. Pose/visibility extraction runs per session (feeds the temporal image compare).

### Storage
- **Postgres + PostGIS** — orgs, projects, **capture sessions (multi-epoch)**, camera poses, visibility, **measurements, snags, snag photos**, annotations, findings; `bim_models`/`alignments` live for the Epic O overlay track. Multi-tenant-ready schema, single-tenant deploy.
- **Object storage (S3 / MinIO)** — raw imagery, ODM artifacts, tiles per session, exports. Signed URLs to the frontend.

### Infrastructure
- Docker; existing WebODM stack reused as reconstruction host
- Topology: small always-on **app VM** + high-CPU/RAM **processing VM** (NodeODM)

---

## Multi-Epoch Consistency (the new first-class requirement)

Cross-session compare and world-anchored snags are only as good as session co-registration. Unlike BIM↔cloud alignment, this is an **operational** problem, not an algorithmic one:

- **Rule:** every capture session of a project is processed with shared GCPs (or RTK) into the **same CRS** (project-level, UTM). Enforced at session creation: a session inherits the project CRS; a mismatch is a validation failure, not a warning.
- **Registration sanity check** (new, cheap pipeline step): after reconstruction, compare per-session GCP residuals and cross-check a handful of stable reference points (GCP markers, permanent structures) across epochs. Emit a per-session `registration_report` (residuals, CRS, GCP set hash). Surfaces silently drifting epochs before the client sees them.
- **No ICP, no feature matching between epochs needed** — if GCP discipline holds, sessions co-register by construction. That is precisely why this scope is deliverable in three weeks and the BIM overlay was not.

---

## Inspection Loop — committed components

### 1. Measurement (WebODM-grade UX)
- **3D:** Potree measurement tools (distance, height, angle, area) with snapping, labels, unit consistency from the project CRS
- **2D:** ortho measurements (distance/area) on the COG via the map view
- Persisted server-side (`measurement` table) per session; listable, exportable to the report
- No new processing machinery — this is frontend polish + a thin CRUD API

### 2. Click-to-image (retained, now per-epoch)
- OpenSfM `reconstruction.json` + tracks → camera poses + visibility per session (unchanged machinery)
- Query: `(project, world_point, session)` → ranked covering images. v3 addition: run the same query **across all sessions** of a project in one call — the backend of the "vs view."

### 3. Cross-session comparison ("time machine")
- **3D compare:** dual-pane Potree, one session per pane, camera state linked client-side (no server involvement; both panes stream their own session's EPT tiles via signed URLs)
- **Image "vs view":** click in either pane → `GET /projects/:id/point-images?point=…&sessions=all` → best image per epoch → side-by-side with synced zoom/pan
- **Session timeline:** `capture_session.captured_at` drives a timeline strip; any two epochs comparable
- Load profile: two Potree instances ≈ 2× tile traffic per client; acceptable single-tenant, note for CDN later

### 4. Snag annotation (inspection, not PM)
- Snag = world-anchored `PointZ` in project CRS + metadata + photo evidence. Because the anchor is world-fixed and sessions are co-registered, a snag renders in **every** epoch — creation epoch, re-inspection epoch, resolution epoch.
- Lifecycle is one flag (`open → resolved`, with `resolved_session_id` capturing *in which epoch* it was verified fixed). No assignment/notification/workflow engine — that line is contractual (roadmap §1/§6).
- Photo evidence: `snag_photo` rows reference (session, image, crop) — the report shows before/after pairs for free.

---

## BIM Registration Engine — planned internal track (not client-committed)

The R1–R4 staged design from the July 4 revision is **unchanged and remains the reference**. It is being executed now as roadmap Epic O (concierge manual path) and productized post-conversion as the wedge:

- **R1** IFC intake & normalization (IFCOpenShell: units, georeferencing, tessellation, GlobalId index)
- **R2** Coarse alignment (georef transform if present; else manual point-pair via CloudCompare SOP — never global ICP first)
- **R3** Fine registration (optional; constrained point-to-plane ICP, Open3D)
- **R4** QA & persistence (versioned `alignment` row: 4×4 transform + method + residuals + correspondences)

The failure-mode table (units mm/m, base-point vs UTM, project-north rotation, global-ICP divergence, bad tessellation) is institutional memory — keep it in this doc even while the track is parked.

**v3 execution posture:** concierge CloudCompare only, ≤2 founder-days pre-gate + ≤2 days integration, gated Jul 13. Output if the gate passes: aligned glTF overlay with toggle/opacity in the viewer (minimal ADR-002 path; swipe compare and element-anchored findings cut), transform + residuals persisted as a versioned `alignment` row. If the gate fails or the timebox overruns: the track **pauses and resumes after the demo** — never discarded; committed scope is unaffected. Client posture: undisclosed until it demonstrably works. `bim_model`/`alignment` tables and the deviation-engine design stay live in the schema.

---

## Visualization Stack

**Committed:** Potree (point cloud, measurement tools) + Three.js (mesh, click-to-image markers, snag markers) + map view for ortho/2D measurements. **Dual-instance Potree** for session compare with client-side camera sync.

**Internal track (Epic O):** BIM glTF/Draco overlay, toggle + opacity only (server-side IFC path per ADR-002; web-ifc still deferred, Option C).

**Deferred:** CesiumJS (portfolio views), swipe/section BIM compare, alignment-QA correspondence view.

---

## Processing Stack

- **WebODM / NodeODM** — reconstruction per session; NodeODM is the integration boundary (`pyodm`, provider ABC)
- **OpenSfM intermediates** — camera poses + visibility per session (click-to-image, vs-view)
- **PDAL** — point-cloud ops + EPT/Potree tiling per session · **GDAL** — COG for ortho/DEM
- *(Epic O track)* **IFCOpenShell** (R1, glTF export) · **Open3D** (transform estimation, ICP) · **CloudCompare** (operator SOP tool)

---

## Aarivix-built components (the intelligence layer)

Reordered for v3 scope:

| # | Component | Enables | Status |
|---|-----------|---------|--------|
| 1 | **Multi-epoch registration discipline + sanity check** | Everything temporal | New, cheap; committed |
| 2 | **Pose + visibility extraction (per session)** | Click-to-image, vs-view | Working; extend cross-session query |
| 3 | **Measurement service + UX** | WebODM-parity measuring | Polish + thin API; committed |
| 4 | **Snag service** | Inspection loop, report | New CRUD + world anchoring; committed |
| 5 | **Tiling / web-optimization** | Cloud/ortho in browser, ×N sessions | Working; committed |
| 6 | **Annotation + findings + report v3** | Leave-behind | Rework around snag register; committed |
| 7 | **BIM Registration Engine (R1–R4)** | Overlay (Epic O), wedge (later) | In MVP as internal track; timeboxed, gated Jul 13, undisclosed to client |
| 8 | **Deviation / findings engine** | Post-conversion wedge | Deferred |

---

## Pipeline (capture-session state machine)

```
Per capture session (unchanged contract, now explicitly per-epoch):
UPLOADED → VALIDATED → QUEUED → RECONSTRUCTING → EXTRACTING → TILING → REG_CHECK → READY
            (CRS/GCP    (NodeODM)      (poses/         (PDAL/     (GCP residuals,
             inherit)                   visibility)      GDAL)      cross-epoch spot check)

Project level:
  sessions[] (co-registered by construction) ─► compare views, vs-view, snags (world-anchored)

Epic O track (internal, gated Jul 13; per bim_model × capture_session):
  PARSE(R1) → COARSE_ALIGN(R2, manual transform injected) → ALIGN_QA(R4) → EXPORT(glTF) → overlay
```

Every stage idempotent. Provider contract (submit/poll/download, status codes 10–50) unchanged; `REG_CHECK` is a post-processing step on our side of the boundary, not a provider change.

---

## Data model (delta from July 4 revision)

- `capture_session` + `captured_at`, `crs`, `gcp_set_hash`, `registration_report jsonb`
- `measurement(id, org_id, capture_session_id, kind, geometry, value_numeric, unit, label, created_by)`
- `snag(id, org_id, project_id, anchor geometry(PointZ), title, note, severity, status, created_session_id, resolved_session_id NULL, created_by, created_at, resolved_at NULL)`
- `snag_photo(snag_id, capture_session_id, image_id, crop_params jsonb)`
- `bim_model`, `alignment` — retained unchanged, populated by the Epic O track; `finding.ifc_global_id` stays nullable until element-anchored findings ship (post-conversion)

---

## API surface (new/changed, DRF)

- `GET/POST /projects/:id/sessions` — timeline (epochs, status, registration report)
- `GET /projects/:id/point-images?point&sessions=all|a,b` — cross-epoch image retrieval (vs-view backend)
- `GET/POST/DELETE /sessions/:id/measurements`
- `GET/POST/PATCH /projects/:id/snags` (+ `?status=`, `?severity=`), `POST /snags/:id/photos`
- `POST /reports/:project_id` — report v3 (snag register, measurement table, compare screenshots)

---

## Planned Features — delivery demo vs later

**Delivery demo (late July, committed):**
- Measurement UX at WebODM parity (3D + ortho)
- Click-to-image (retained)
- Cross-session compare: dual-pane 3D + image vs-view + timeline
- Snag annotation with open/resolved lifecycle and photo evidence
- Report v3: snag register with before/after pairs, measurements, honest accuracy statement

**Internal track (in MVP, not client-committed, gated Jul 13, undisclosed until working):**
- Manually aligned CAD/BIM overlay, toggle + opacity

**Later (pull-based, post-conversion — roadmap §6):**
- Assisted/automated BIM alignment + element-anchored deviation findings (the wedge, revived on revenue)
- Automated change detection between epochs
- Snag workflows (assignment, notifications, dashboards) — only on named customer pull
- web-ifc semantics · CesiumJS · multi-tenant deploy · volume/cut-fill

---

## Scale — explicitly deferred

Unchanged: multiple/GPU NodeODM nodes, webhooks over polling, CDN for tiles (note: dual-pane compare doubles tile egress — first thing to revisit if usage grows), multi-tenant isolation/quotas, k8s end-state, automated registration at scale.

---

## High-level diagram

```
Frontend (Next.js)  ──►  Aarivix API (Django + DRF)  ──►  Aarivix DB (Postgres+PostGIS)
 Potree ×2 (compare)        │   sessions(multi-epoch) · poses · visibility
 + Three.js (snags,         │   measurements · snags · snag_photos · findings
   click-to-image)          │   [bim_models · alignments — Epic O track]
 + ortho map (2D measure)   │
                            ├─► Celery (Redis) ─► Aarivix worker (per session)
                            │     validate(CRS/GCP) → NodeODM → poses/visibility
                            │     → tile → REG_CHECK(residuals, cross-epoch)
                            │     [Epic O: R1 parse → R2 manual → R4 QA → glTF]
                            ├─► NodeODM (pyodm, direct)        [reconstruction]
                            ├─► PDAL / GDAL                    [tiling]
                            └─► Object storage (S3/MinIO)      [artifacts + tiles/epoch → signed URLs]

 CloudCompare (operator tool, Epic O track): manual alignment → transform import.
 WebODM webapp = optional ops UI only.
```

---

*Updated July 5, 2026 for MVP Roadmap v3 (reduced committed scope; client compromise). Decision records: ADR-001, ADR-002. Protected contracts: `docs/reference/pipeline-contract.md`, `docs/reference/product-spec.md`. The July 4 BIM Registration Engine design is retained above as the Epic O / wedge reference — do not delete it; it is institutional memory. Suggested follow-up ADR-003 (revised): "Multi-epoch co-registration discipline and the REG_CHECK stage."*
