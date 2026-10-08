# Aarivix — MVP Roadmap v3: The Inspection Compromise

> **Supersedes v2 (the BIM-overlay wedge, July 4 2026).** Rewritten July 5, 2026 after reaching a scope compromise with the pilot client. Companions: `Aarivix_System_Architecture.md` (updated in lockstep), `Aarivix_Pilot_Spec.md` (v1, historical), `construction_reality_intelligence_project.md` (GTM context).
>
> **Founders:** Chinmay Gandevikar (frontend, 3D/visualization) · Nirnay Roy (backend, ML, reconstruction)

---

## 0. The one-paragraph version

The client and we have agreed on a reduced, deliverable scope: **a polished measurement experience (WebODM-grade), click-to-image, and cross-session time comparison so their team can manually inspect for snags and annotate them.** The CAD/BIM overlay — v2's entire critical path — is now a **planned internal track (Epic O)**: we build a manually aligned overlay inside the MVP, but it is not committed to the client, not disclosed until it works, and the deal does not hinge on it. This trades v2's high-risk/high-moat bet for a low-risk committed delivery: every committed feature builds on components that already work (viewer, click-to-image, reconstruction pipeline), and the one genuinely new capability — session-over-session comparison — rides on GCP/RTK georeferencing we already do, not on the unsolved BIM alignment problem.

---

## 1. What changed and why

| Question | Answer |
|---|---|
| What did the client agree to? | Committed: measurement UX, click-to-image, time/vs comparison for manual snag inspection, snag annotation. Additionally, we build a **manually aligned CAD overlay as a planned internal track** — not committed to the client and not disclosed until it works. |
| What does this change from v2? | The overlay moves from *the* critical path to an optional parallel track. The Jul 13 go/no-go gate is no longer a deal gate — it decides only whether the overlay appears in the demo or the track pauses until after it. |
| Is this the PM pivot we rejected? | **No — and we hold that line.** Snag *annotation* (mark, describe, photo-evidence, open/resolved) is inspection tooling on the twin. Snag *workflows* (assignment, notifications, dashboards, cross-department collaboration) remain deferred and pull-based (§6). The moment we build assign-to-user, we're competing with Powerplay on distribution we don't have. |
| Why is this scope low-risk? | Click-to-image and the viewer already work. Measurement exists (needs polish to WebODM grade). Cross-session comparison needs sessions in a common spatial frame — GCP/RTK georeferencing gives us that nearly free, unlike BIM↔cloud alignment. |
| What's the strategic cost? | The overlay was the differentiator and the client's original stated blocker. We accept slower moat-building in exchange for a converted, referenceable customer. The wedge thesis isn't dead — it's now funded by revenue instead of blocking it. |

---

## 2. Timeline

Anchored on today (Sun **Jul 5, 2026**). The delivery demo replaces v2's re-demo; conversion window unchanged.

| Date | Milestone |
|---|---|
| **Jul 6–8** | **Dependency check (hard):** confirm a **second capture session** of the client site exists or schedule the flight now — time-compare cannot be demoed on one epoch. Confirm both sessions share GCPs/CRS. |
| **Jul 6–17** | **Phase 1 — committed build.** Measurement polish (Epic K), cross-session compare (Epic L), snag annotation (Epic M). |
| **Jul 13** | **Overlay gate:** alignment spike verdict (v2 Epic G checklist, ≤2 founder-days pre-gate). Pass → integration proceeds (Epic O), overlay appears in the demo as an unannounced bonus. Fail → written diagnosis, track pauses until after the demo; committed scope unaffected either way. |
| **Jul 18–24** | **Phase 2 — hardening + report.** Report v3, demo rehearsal on the client's own two epochs, bug triage. |
| **Last week of July** | **Delivery demo** to the client on their data. Pricing sheet on the table. |
| **Mid-August** | Conversion window closes: signed agreement or a diagnosed reason why not. |

**Parallel sales track (unchanged):** keep working `Aarivix_Pilot_Leads.csv`. The pitch is now "digital twin + time machine + snag inspection" — more demoable to cold leads than the overlay was, because it needs only their imagery, not their BIM. Target: 5+ conversations before the demo.

---

## 3. Current build state (as of Jul 5)

**Working (all committed scope builds on these):**
- Upload → NodeODM → LAZ reconstruction (`pyodm`, provider-abstracted — `docs/reference/pipeline-contract.md`)
- 3D / point-cloud viewer (Potree + Three.js)
- Click-to-image inspection (retained hero)
- Basic measurement + annotation (needs UX polish to WebODM grade)
- Concierge findings + report workflow
- Auth/org/projects — `docs/reference/product-spec.md` flags auth as not wired; reconcile in Phase 1

**New in committed scope:**
- Measurement UX at WebODM parity (Epic K)
- Multi-session projects + cross-session comparison, 3D and image (Epic L)
- Snag annotation with lifecycle (Epic M)

**Internal track (planned, not client-committed):**
- Manually aligned CAD overlay — v2 Epics G/H survive as the timeboxed Epic O; we build it, we just don't promise it

---

## 4. Phase 1 — committed build (Jul 6–17)

### Epic K — Measurement UX to WebODM grade [C]
WebODM's measurement experience is the client's stated reference. Parity checklist:
- **3D (Potree tools):** distance, height, angle, area; point/edge snapping; per-measurement labels; unit display (m/cm) consistent with CRS
- **2D (ortho):** distance + area on the orthophoto view (Leaflet/OpenLayers path), synced to the same project
- Measurement persistence: saved per capture session, listable, deletable, exportable into the report
- Polish bar: no floating-label glitches, mobile-tolerant hit targets, undo
- *Acceptance:* a site engineer measures a slab edge and a floor area in <30 s each without instruction, and the numbers match tape/known dimensions within survey accuracy.

### Epic L — Time machine: cross-session comparison [C viewer, N registration + image retrieval]
The genuinely new capability. Two modes, one principle: **all sessions of a project live in one spatial frame** (GCP/RTK georeferenced, same CRS — verify, don't assume; Epic L starts with a registration sanity check on the client's two epochs).
- **3D compare:** dual-pane synced Potree viewers (linked camera), session A vs session B; datetime picker per pane
- **Image compare ("vs view"):** click a point in either pane → best covering photo from *each* session (existing pose/visibility machinery, run per session) → side-by-side images with synced zoom/pan. This is click-to-image made temporal — the inspection money shot.
- Session picker UX: project timeline strip; any two epochs comparable
- *Acceptance:* client picks a location, sees May-photo vs July-photo of the same spot side by side in <5 s, and can spot a real construction change.

### Epic M — Snag inspection + annotation [C front, N backend]
Inspection tooling, not PM software (see §1 line 3).
- Snag = **world-anchored 3D point** + title, note, severity, photos (auto-attached from click-to-image), creator, timestamps
- Because snags are anchored in world coordinates, they render in **every** session — created in the July epoch, checkable against the September epoch. Lifecycle: `open → resolved` (one flag, no workflow engine).
- Snag list panel: filter by status/severity, click → fly-to in viewer, appears in report
- *Acceptance:* client's engineer walks the twin, logs 5 snags with photo evidence in one sitting, and re-inspects them in compare view.

### Deliberately thin
- No assignment, notifications, or dashboards (deferred, §6)
- No automated change detection between sessions — the human is the change detector; we supply the viewport

---

## 5. Phase 2 — hardening + report (Jul 18–24)

### Epic N — Report v3 + demo kit [either]
- Leave-behind PDF rebuilt around inspection: snag register (photo pairs across time, status), measurement table, session-compare screenshots
- If Epic O passed its gate: overlay screenshots included, labeled "preview"
- Pricing sheet ready on demo day; retain the "if the report doesn't surface 3 things you didn't know, you don't pay" framing
- *Acceptance:* the PDF sells the inspection loop when we're not in the room.

### Epic O — Concierge CAD overlay [planned internal track; timeboxed; not client-committed]
This is part of the MVP plan — we are building it — but it is deliberately shielded from client expectations.
- **Scope:** exactly v2 Epic G (CloudCompare manual SOP, §4 diagnosis checklist) + minimal Epic H (aligned glTF overlay in the viewer, toggle/opacity — swipe compare cut). Transform + residuals persisted as the versioned `alignment` artifact.
- **Timebox:** ≤2 founder-days pre-gate (Jul 13), ≤2 days integration post-gate. Overruns pause the track rather than extend it — committed scope owns the calendar. If paused, it resumes immediately after the demo; the work is never discarded (it is the §6 wedge's on-ramp).
- **Client posture: do not mention the overlay until it demonstrably works.** No expectations set, no repeat of the pilot. It either appears in the demo as an unannounced bonus or stays internal.
- *Acceptance (gate):* pilot IFC visibly aligned to the pilot cloud in our viewer; transform + residuals persisted; written diagnosis of the June failure; SOP hours logged (feeds future pricing).

---

## 6. Deferred — pull-based, post-conversion

Rule unchanged: **nothing below gets built until a paying customer asks for it by name.**

- Committed/automated **BIM overlay + alignment** (assisted in-viewer picking → automated registration) — the v2 wedge thesis, revived on revenue
- Element-anchored deviation findings + automated deviation engine
- Automated change detection between sessions (diff heatmaps)
- Snag **workflows**: assignment, notifications, dashboards, integrations — the PM-pivot line stays where v2 drew it
- Multi-tenancy, CesiumJS portfolio views, web-ifc semantics (ADR-002 Option C), volume/cut-fill, hydrology

---

## 7. Data model delta

Base schema per v2 §7 (`org_id`-everywhere unchanged). v2's `bim_model`/`alignment` tables **stay in the schema** (populated by Epic O; future wedge) but drop off the critical path.

New/changed:
- `capture_session` — already exists; add `captured_at` (epoch date, drives the timeline strip) and enforce shared CRS per project
- `snag(id, org_id, project_id, anchor geometry(PointZ), title, note, severity, status[open|resolved], created_session_id, resolved_session_id NULL, created_by, created_at, resolved_at NULL)`
- `snag_photo(snag_id, capture_session_id, image_id, crop_params jsonb)` — evidence pairs across epochs
- `measurement(id, org_id, capture_session_id, kind[distance|height|angle|area|area2d|distance2d], geometry, value_numeric, unit, label, created_by)`
- `finding.ifc_global_id` stays nullable-and-unused until the overlay ships for real

---

## 8. Risks & mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| **No second capture epoch of the client site** | **High — silent killer** | Jul 6–8 dependency check is the first action of Phase 1. If missing, schedule the flight immediately; time-compare on a single epoch is undemoable. |
| Sessions don't co-register (GCP mismatch, CRS drift) | Med | Registration sanity check opens Epic L; fix is operational (shared GCPs), not algorithmic. |
| Scope creep back toward PM software ("can it also assign to the contractor?") | Med | §4 "deliberately thin" list is the contract; answer is "post-conversion, happily." |
| Overlay track eats committed calendar | Med | Hard timebox in Epic O; overruns pause the track (resume post-demo), committed scope owns the calendar by rule. |
| Client reads reduced scope as reduced value → price pressure | Med | Price the inspection loop (recurring per-epoch capture + report), not the feature list; time-compare value *grows* with each capture. |
| Measurement accuracy challenged on site | Low-Med | GCP/RTK residuals in the report (v2 discipline retained); state accuracy honestly. |
| Two founders, three weeks | Med | Epics K/M are C-heavy, L splits cleanly, N is shared; O is the only pausable track. |

---

## 9. Open questions

1. **Does a second epoch of the client site exist?** If yes: captured when, with the same GCPs? If no: flight date? (Blocks Epic L acceptance.)
2. **Pricing model** — per-epoch capture+report subscription fits the time-machine value curve better than one-off scans. Decide before the demo.
3. **Who runs inspection at the client** — the Projects Head or a site engineer? That person drives the demo; snag UX should fit *their* hands.
4. **What does the client mean by "measurements like WebODM"** — have they used WebODM's ortho measurements, 3D tools, or both? Ask; it scopes Epic K's 2D work.
5. **Overlay track:** did the failed June alignment attempt ever get its Day-1 diagnosis (v2 §9 Q1)? Two days of Epic O budget go further with that answer in hand.

---

## 10. Next steps (this week)

1. Confirm/schedule the second capture epoch (Epic L blocker — do this first)
2. Ask the client the WebODM-measurement question (§9 Q4) and identify the demo driver (§9 Q3)
3. Start Epic K measurement polish; run the Epic L registration sanity check as soon as two epochs exist
4. Timeboxed Epic O Day-1 diagnosis on the failed alignment
5. Continue leads outreach: 3 calls from the CSV, pitching the inspection loop

---

*Rewritten July 5, 2026 after the client scope compromise. v2 (BIM-overlay wedge, July 4) is superseded but its alignment work survives as the Epic O internal track and the §6 revival path. Architecture updated in lockstep: `Aarivix_System_Architecture.md`.*
