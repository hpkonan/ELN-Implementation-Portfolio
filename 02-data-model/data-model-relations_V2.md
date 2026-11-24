# Simple Western data model – relationship documentation

ELN portfolio project (eLabFTW) · version 2.3, November 24, 2026 · fictional data

This document accompanies the diagram `eln-simple-western-data-model_V2.drawio`. It describes a **conceptual model**: how the lab actually works, independently of any software. The last sections explain how each element is implemented in eLabFTW.

## 1. Objective

Starting from any result, be able to answer three questions:
1. **With what** was it obtained? (antibody lots, module lots, instrument)
2. **On which sample**, prepared how? (condition, extraction, preparation)
3. **What changed** between two runs that give different results?

## 2. The five entity families

| Family | Entities | Role |
|---|---|---|
| Products | Primary antibody, Separation module, Detection module | The catalog: characteristics that do not change |
| Physical lots and instrument | Primary antibody lot, Separation module lot, Detection module lot, Instrument | The actual tubes, kits and machines |
| Sample chain | Biological condition, Extraction, Sample preparation | From the phenotype to what is loaded into the plate |
| Execution | Experiment | The Jess run, with its Compass plate layout attached |
| Results | Result | What was measured, one record per peak, with the Compass export columns |

## 3. Keys

| Key | Role | Example |
|---|---|---|
| PK (technical key) | Number assigned automatically by the database; unique, never changed or reused | `id=41` in the eLabFTW page address |
| UK (unique readable code) | Name used by people; unique, can be corrected | SM-FL004, SM-FL004 · lot P-1187, JS-5104 |
| FK (reference) | Points to the PK of another record | The lot points to module no. 3 |

When an official identifier already exists, it is used directly as the UK instead of an invented code: catalog number for modules, vendor lot number for lots, serial number for the instrument.

For lots, a vendor lot number is only unique for a given product. The readable code therefore combines the product code and the lot number: `SM-FL004 · lot P-1187`. The link to the product remains the source of truth; the prefix helps people read the record and pick the right lot when creating a link.

## 4. Relationships

How to read: "1 → 0..n" means that one object on the left is linked to zero, one or several objects on the right, and that each object on the right is linked to exactly one object on the left. "n ↔ n" means several on each side.

| From | To | Cardinality | Business meaning |
|---|---|---|---|
| Primary antibody | Primary antibody lot | 1 → 0..n | An antibody may have been received in several lots |
| Separation module | Separation module lot | 1 → 0..n | Same for separation modules |
| Detection module | Detection module lot | 1 → 0..n | Same for detection modules |
| Biological condition | Extraction | 1 → 0..n | The same phenotype can be extracted several times (biological replicates) |
| Extraction | Sample preparation | 1 → 0..n | The same lysate is diluted again for each experiment |
| Experiment | Sample preparation | 1 → 0..n | Samples are prepared fresh for a given run |
| Instrument | Experiment | 1 → 0..n | Each run is performed on one instrument |
| Separation module lot | Experiment | 1 → 0..n | Each run uses one separation module lot |
| Primary antibody lot | Experiment | n ↔ n | A run uses one or more antibody lots; a lot is used in several runs |
| Detection module lot | Experiment | n ↔ n | Same for detection modules (one per channel in multiplex runs) |
| Experiment | Experiment | 0..1 → 0..n | A rerun or replicate points to its original experiment |
| Experiment | Result | 1 → 0..n | A run produces one result per detected peak |

## 5. Design decisions

| # | Decision | Business rationale |
|---|---|---|
| 1 | One experiment = one Jess run | The raw file and consumables are attached to the run without ambiguity |
| 1a | Run type (first run, technical rerun, biological replicate) + link to the original experiment | Reruns and replicates can be compared directly |
| 2 | Traceability at run level: lots are linked to the experiment, and the Compass plate layout states what is in each capillary | Capillaries have no useful attributes of their own; the plate layout annotated in Compass is the reference |
| 3 | Plate layout = CSV file exported from Compass, attached to each experiment | The layout is filled in Compass and re-imported for a new run; no dedicated record |
| 4 | Three-level sample model: condition → extraction → preparation | When the signal is poor, solubilization, extraction, denaturation, dilution and deglycosylation are investigated |
| 4a | Fresh preparation for each run, linked to the experiment | Samples are diluted again for each experiment |
| 4b | The stock lysate concentration is recorded only once, in the extraction | The preparation retrieves it through the link; no double entry |
| 5 | The result reuses the Compass export columns, with their names: Sample, Sample Attr, Primary, Primary Attr, Secondary, Cap, Name, MW (kDa), Height, Area, Corr. Area, S/N, Baseline, Channel | The script copies them without a translation table; users recognize their export; no Capillary entity |
| 6 | Several peaks may be expected: one record per peak | Isoforms, phosphorylated forms, multiplexing |
| 7 | Normalization = a run setting (none, total protein, housekeeping protein) plus where it was calculated (Compass or Excel), recorded in the experiment. The Corr. Area column stays empty when the run is not normalized. If calculated in Excel, the workbook is attached to the experiment | Compass does the calculation, or Excel for complex experiments; the ELN records how the run was normalized and keeps a trace of any external calculation. No more result-to-result link: a simpler diagram |
| 8 | Re-analysis in Compass: results are replaced; the Compass Analysis log can be attached (optional) | Compass already records every change with date and time |
| 9 | Links between records are the source of truth; the readable code is a unique label | Correcting a code never breaks traceability |
| 10 | Official identifiers as UK (catalog number, vendor lot number, serial number) | Fewer codes to invent, and the same name on the label and in the ELN |
| 11 | No lot status; no remaining quantity, except optionally for separation module lots | Only document what the lab actually uses |
| 12 | Readable code of a lot = product code · vendor lot number | A lot number is only unique per product; when linking, eLabFTW lists records by title |
| 13 | Run settings (antibody diluent / blocking buffer, default or modified protocol, changes made) are attributes of the experiment, not entities | They describe a single run; the full detail is in the attached Compass run file. The diagram stays readable for stakeholders |
| — | Preparation reagents (deglycosylation enzyme) as text attributes | Traceable without adding more entities |

## 6. Example traceability questions

**"In which experiments was separation module SM-FL004, vendor lot P-1187, used?"**
Separation module lot → Experiment (direct links).

**"Why does replicate 2 give a weaker signal than replicate 1?"**
Compare the two experiments connected by the "origin" link: lots used (Experiment), then preparation, extraction and condition (Experiment → Sample preparation → Extraction → Condition).

**"Is this result normalized, and to what?"**
Result → Experiment → Normalization fields; the detail is in the Compass file or the attached Excel workbook.

**"Could this run detect a 250 kDa protein?"**
Experiment → Separation module lot → Separation module → size range (12–230 kDa): no.

## 7. Implementation in eLabFTW

| Model element | eLabFTW implementation |
|---|---|
| Products, lots, instrument | Resource categories, templates with structured fields, resource → resource links |
| Condition, extraction, preparation | Resource categories linked to each other; the preparation is also linked to the experiment |
| Experiment | Experiment created from the Simple Western template, linked to the lots used |
| Plate layout | CSV exported from Compass, attached to the experiment |
| Result | Results table in the experiment, with the Compass export columns in the same order, imported by the integration script |
| Links as the source of truth | "Linked resources" section (eLabFTW internal identifier) |
| Rules (unknown lot, missing field) | Validation in the script: required fields in eLabFTW are only indicative |
| Uniqueness of readable codes | Not enforced by eLabFTW: usage rule (search before creating) and check by the script |

**Main limitation:** lots are linked to the run, not to the capillary. In the rare case of a run comparing two lots of the same antibody, knowing which lot was in which capillary requires reading the Compass plate layout. This is a deliberate simplicity trade-off, to be documented in the design note.

## 8. Out of scope

- Automatic tracking of consumed quantities.
- Lot-specific antibody concentration: available on the vendor datasheet and rarely used; more fields would bury the useful information.
- Typed capillary roles (ladder, controls).
- Lot traceability at capillary level.
- Calculating normalization in the ELN (done by Compass or in Excel).
- Result versioning in the ELN (history relies on Compass and its Analysis log).
