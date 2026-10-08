# AarivIx System Architecture

## Current Development Architecture

### Frontend
- Next.js
- React
- TypeScript
- TailwindCSS

### Backend
- Django
- Django REST Framework

### Infrastructure
- Docker
- Existing WebODM stack

---

# Existing WebODM Services

## Containers

| Container | Purpose |
|---|---|
| webapp | WebODM application |
| worker | processing worker |
| broker | Redis queue |
| db | PostgreSQL database |
| webodm_node-odm_1 | NodeODM processing |

---

# Current Strategy

AarivIx will initially:

- orchestrate around WebODM
- extend workflows
- add construction monitoring intelligence
- provide improved visualization
- provide BIM comparison tools
- provide reporting and dashboards

AarivIx will NOT initially replace ODM processing internals.

---

# Planned Visualization Stack

- CesiumJS
- Potree
- Three.js

---

# Planned Processing Stack

- WebODM
- NodeODM
- GDAL
- PDAL

---

# Planned Features

- Orthomosaic visualization
- Point cloud visualization
- DEM visualization
- BIM comparison
- Construction progress tracking
- Annotation systems
- Measurements
- Reporting
- Inspection workflows