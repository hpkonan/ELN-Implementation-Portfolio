# ELN Implementation Portfolio – Simple Western Workflow

A hands-on project to model, configure and integrate a real laboratory workflow in an Electronic Lab Notebook (ELN), from requirements to data integration, using the open-source ELN eLabFTW.

**Status:** In progress (started October 2026)
**Type:** Personal learning project – all data is fictional

---

## Why this project

I am a PhD-trained life scientist with several years of experience supporting automated protein analysis platforms in biopharma and academic labs. I know the bench side of lab workflows well; this project is about the digital side: how a workflow is structured, captured and connected inside lab informatics systems.

The goal is to go through the full cycle a Solutions Architect, Solution Consultant or Business Analyst would follow on an ELN project:

1. Understand the scientific workflow and the users' needs
2. Model the data (samples, reagents, instruments, results)
3. Configure the ELN (resources, templates, statuses)
4. Integrate instrument data through an API
5. Plan the deployment in a real lab (hosting, backups, access, responsibilities)
6. Document requirements, user stories and acceptance testing

## The scenario

A fictional biotech characterizes antibody candidates using **Simple Western** (capillary-based automated Western blot). Today, results are scattered across instrument exports, Excel files and slide decks. Nobody can easily answer: *"Which experiments used antibody lot X, and what were the results?"*

This project builds the structure that makes that question answerable.

## Tools

| Tool | Use |
|---|---|
| [eLabFTW](https://www.elabftw.net) | Open-source ELN: experiments, resources, templates, audit trail |
| Docker Compose | Running eLabFTW, MySQL and the reverse proxy as containers |
| [Caddy](https://caddyserver.com) | HTTPS reverse proxy in front of eLabFTW |
| draw.io | Data model (ERD) and architecture diagrams |
| Python + eLabFTW REST API v2 | Importing instrument results into experiments *(planned)* |

## Roadmap

- [x] Explore eLabFTW: experiments vs. resources, linking, revision history
- [x] Self-host eLabFTW 6 locally (Docker Compose, MySQL, Caddy HTTPS reverse proxy)
- [ ] Set up the team, users and roles
- [x] Design the conceptual data model (ERD in draw.io + relationship documentation)
- [ ] Configure eLabFTW to match the data model: resource categories, templates, sample chain, Simple Western experiment template
- [ ] Test traceability end to end: first run, technical rerun, traceability questions
- [ ] Data integrity review: audit trail and revisions mapped to ALCOA+ principles
- [ ] Python integration: CSV instrument export → validation → eLabFTW via API
- [ ] Solution architecture diagrams: integration flow with error handling, optional n8n orchestration, lab deployment
- [ ] Business analysis deliverables: discovery guide, user stories, UAT plan, build-vs-buy note
- [ ] Demo video and final write-up

## Repository structure *(will grow as the project progresses)*

```
.
├── README.md               ← overview and learning log
├── 00-infrastructure/      ← docker-compose.yml and Caddyfile (no secrets)
├── 01-exploration/         ← notes and screenshots from the first sessions
├── 02-data-model/          ← ERD (draw.io + PNG) and relationship documentation
├── 03-eln-configuration/   ← resource categories, templates, screenshots
├── 04-integration/         ← Python script and sample data
├── 05-architecture/        ← integration flow and lab deployment diagrams
└── 06-business-analysis/   ← discovery guide, user stories, UAT plan
```

---

## Learning log

### Session 1 – First contact with an ELN (October 2026)

Worked on the public eLabFTW demo instance.

**What I learned**

- **Experiments vs. resources.** An ELN separates *what you did* (experiments: objective, protocol, results) from *what you used* (resources: antibodies, samples, consumables, instruments). This separation is what makes traceability possible.
- **Linking is the key feature.** Linking an experiment to an antibody resource (via the linked-items section or by typing `#` in the editor) creates a two-way relationship: from the antibody page, I can see every experiment that used it. This is impossible with a paper notebook.
- **Revisions are not a full audit trail.** The demo server stores a maximum of 10 revisions per entry, and a change of at least 100 characters is required to trigger a new revision. A small edit, such as changing a dilution from 1:50 to 1:100, may not create a revision at all. Both settings are configured by the server administrator.

**Open questions for next sessions**

- Does eLabFTW keep a separate, field-level change log in addition to revisions?
- How would these revision settings need to be configured to meet GxP data integrity expectations?
- How can resource types be standardized with structured fields?

### Session 2 – Self-hosting eLabFTW 6 (October 2026)

Installed my own eLabFTW instance on a personal laptop, to get admin rights and API access that the public demo does not provide.

**Architecture**

```
Browser ──HTTPS──► Caddy (reverse proxy) ──HTTP──► eLabFTW 6 (web app) ──► MySQL 8.4
```

Three containers orchestrated with Docker Compose, with persistent Docker volumes for the database and uploaded files. Configuration files are in `00-infrastructure/` (secrets removed).

**Issues I diagnosed from the container logs**

| Symptom | Root cause | Fix |
|---|---|---|
| Browser: connection closed | v6 requires a TLS certificate path in HTTPS mode | Run the app in HTTP mode behind a reverse proxy |
| `[pool elabpool] user has not been defined` | Configuration written for v5; v6 runs as a non-root user | Added `user`, a `tmpfs` for `/run`, new uploads path and container port 8080 |
| `can't create directory '/var/cache/elabftw/': Read-only file system` | v6 runs on a read-only filesystem | Added a writable `tmpfs` for the cache |
| "eLabFTW works only in HTTPS" banner | The app refuses plain HTTP | Added Caddy as an HTTPS reverse proxy; it forwards `X-Forwarded-Proto: https` |

**What I learned**

- **Read the logs first.** Every blocking issue was stated explicitly in the container logs (`docker logs`). Guessing from the browser error alone would have been much slower.
- **Major versions change deployment, not just features.** A configuration that worked for v5 broke in four places on v6. Checking the upgrade guide before installing would have saved time; this matters when planning a client upgrade.
- **Security is built into the deployment.** eLabFTW 6 runs as a non-root user, on a read-only filesystem, and only over HTTPS. These are the controls an IT or QA team expects to see for a system that holds lab records.
- **Test vs. production.** This local setup uses a self-signed certificate and fictional data. A production deployment (server, certificate from a trusted authority, backups, monitoring) is a different scope.

**Update – enabling an add-on.** I later enabled the optional OpenCloning add-on (molecular biology tools). It runs as a fourth container on the internal network, with no port exposed to the host, and is switched on with two environment variables in the eLabFTW service. Not needed for a protein workflow, but a good exercise in how optional modules are deployed.

### Session 3 – Designing the data model (November 2026)

Designed a conceptual data model of the Simple Western workflow before touching the eLabFTW configuration, then reviewed it against how labs actually work.

![Simple Western data model](02-data-model/eln-simple-western-data-model_V2.png)

**Scope of version 2**

- **12 entities in 5 families:** products (primary antibody, separation module, detection module), physical lots and instrument, sample chain (biological condition → extraction → sample preparation), experiment (one Jess run), results (one record per peak).
- **12 relationships,** including many-to-many links between experiments and antibody or detection module lots, and a self-reference: a rerun points to its original experiment.
- **Results mirror the instrument export:** one record per peak, with the Compass column names (Sample, Primary, Cap, MW, Area, Corr. Area, S/N, Channel…), so the integration script needs no translation table.
- **13 documented design decisions** and the traceability questions the model must answer. Full documentation: [`02-data-model/data-model-relations_V2.md`](02-data-model/data-model-relations_V2.md).

**What I learned**

- **Model the lab, not the tool.** Designing the model independently of eLabFTW made the gaps visible: the model can describe what happens in each capillary, but eLabFTW links records at experiment level. Documenting that gap is part of the deliverable.
- **Three kinds of keys, three jobs.** The technical key (PK) is assigned by the database and never changes; links rely on it, so correcting a readable code never breaks traceability. The readable code (UK) is what people use, and it stays the same across systems, which is what makes a migration verifiable. The reference (FK) is the link itself.
- **Reuse official identifiers.** Catalog numbers, vendor lot numbers and serial numbers serve as readable codes. They are already unique, already on the label, and there is nothing to invent.
- **Simplicity is a design decision.** Version 1 had 16 entities and traced every lot down to the capillary. Reviewed against real practice (the plate layout lives in Compass, capillary numbers are in every export), version 2 keeps 12 entities and traces at run level. The trade-off is written down: in the rare run comparing two lots of the same antibody, the plate layout tells which lot was in which capillary.
- **Run settings are attributes, not entities.** Antibody diluent, protocol changes and normalization describe one run. They are fields of the experiment; the attached Compass file (or the Excel workbook, when normalization is calculated outside Compass) holds the detail.
- **Only capture what the lab uses.** Fields such as lot status or remaining quantity were removed where nobody would maintain them. A field nobody fills in becomes a source of false information.

**Next step:** align the eLabFTW configuration with this model (resource categories, templates, sample chain, experiment template), then test it with a first run and a technical rerun.

---

## Notes

- **All data in this project is fictional.** No real customer, company or patient data is used.
- This is an independent learning project. It is not affiliated with, endorsed by or developed for any company or software vendor mentioned here.
- Some parts were drafted with AI assistance; all content was reviewed, tested and validated by me.
