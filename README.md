# Off-Grid Schools Identification

**Project file:** `Off_Grid_Schools_Identification.qgz`
**Coordinate reference system:** EPSG:32628 (WGS 84 / UTM Zone 28N)
**Study area:** Sierra Leone
**Date completed:** May 2026

---

## Purpose

Access to electricity in educational facilities is a critical enabler of quality learning, yet a significant proportion of schools across Sierra Leone remain beyond the reach of the national medium voltage grid. This project identifies those facilities by comparing the spatial extent of the MV distribution network against the locations of education facilities across the country. Schools falling outside a 500 m grid proximity threshold are flagged as off-grid candidates, producing a spatially accurate, decision-ready dataset that energy planners, government ministries, and development partners can use to prioritise and site off-grid solar interventions where they are needed most.

---

## Input layers

| Layer | File | Geometry | Features | CRS |
|---|---|---|---|---|
| Education facilities | `edu_facilities` (source dataset) | Point | 475 | EPSG:32628 |
| MV distribution lines | `mv_lines` (source dataset) | Line | 3,739 | EPSG:32628 |
| Administrative districts | `SLE_adm2` (source dataset) | Polygon | 14 | EPSG:32628 |

All layers were confirmed in EPSG:32628 before any processing began.

---

## Processing workflow

### Step 1 - MV line buffer (500 m on-grid corridor)

A 500 m fixed-distance buffer was generated around all MV line features using **Processing Toolbox > Native > Buffer**, with dissolve enabled to produce a single unified on-grid corridor polygon. The 500 m threshold represents a practical electrification proximity standard used by energy access practitioners. The output was saved as `mv_lines_500m_buffer.gpkg`, containing one dissolved polygon feature.

### Step 2 - Grid access tagging (Join Attributes by Location)

Each education facility was evaluated for whether it falls within the 500 m MV buffer using **Processing Toolbox > Native > Join Attributes by Location**. The join appended MV line attributes (nominal voltage, operating voltage, conductor type, conductor size, conductor resistance) to each school point that intersects the buffer. A `grid_access` text field was then computed via the field calculator:

| Value | Condition |
|---|---|
| `On-Grid` | School falls within `mv_lines_500m_buffer.gpkg` |
| `Off-Grid` | School falls outside the buffer |

The resulting layer `edu_facilities_grid_tagged.gpkg` retains every education facility feature from the input. Total feature count is preserved at 475. This layer is the primary analytical deliverable.

Intermediate working copies produced during development are stored as `_off_calc.gpkg`, `_on_calc.gpkg`, and `schools_grid_join.gpkg`.

An additional layer `edu_facilities_grid_status.gpkg` represents a separate classification pass retained for reference.

### Step 3 - Subset extraction

Two filtered subsets were exported from `edu_facilities_grid_tagged.gpkg` using **Processing Toolbox > Native > Extract by Attribute**:

- `schools_off_grid.gpkg` — features where `grid_access = 'Off-Grid'`
- `schools_on_grid.gpkg` — features where `grid_access = 'On-Grid'`

### Step 4 - Symbology

In the QGIS project, `edu_facilities_grid_tagged.gpkg` is symbolised using a categorised renderer on the `grid_access` field:

| Value | Symbol | Colour |
|---|---|---|
| `On-Grid` | Circle | Green |
| `Off-Grid` | Circle | Red |

`SLE_adm2` is displayed as the base administrative boundary reference layer.

---

## Output layers

| File | Features | Description |
|---|---|---|
| `mv_lines_500m_buffer.gpkg` | 1 | Dissolved 500 m on-grid corridor |
| `edu_facilities_grid_tagged.gpkg` | 475 | Primary deliverable - all schools with `grid_access` field |
| `schools_off_grid.gpkg` | 467 | Schools classified `Off-Grid` |
| `schools_on_grid.gpkg` | 8 | Schools classified `On-Grid` |
| `schools_grid_join.gpkg` | - | Intermediate spatial join output |
| `edu_facilities_grid_status.gpkg` | - | Alternative classification pass |

---

## Key findings

- 475 education facilities were assessed across Sierra Leone.
- 467 schools (98.3%) fall beyond 500 m of the MV network and are classified Off-Grid.
- Only 8 schools (1.7%) fall within the 500 m on-grid corridor.
- The MV attributes joined to On-Grid schools indicate nominal voltages of 33 kV, consistent with the distribution tier of the Sierra Leone national grid.
- The 467 off-grid schools represent the primary candidate list for off-grid solar photovoltaic interventions under any national school electrification programme.

---

## Notes

- `edu_facilities_grid_tagged.gpkg` has only 12 features loaded in the current project view; this reflects a filtered display configuration in the saved project rather than a reduction of the underlying layer, which holds all 475 features. The complete dataset is intact in the file and confirmed by the `schools_off_grid.gpkg` (467) and `schools_on_grid.gpkg` (8) counts summing to 475.
- Task reference document: `Task1.docx`.
