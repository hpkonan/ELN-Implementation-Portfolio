# ELN Implementation Portfolio – Simple Western Workflow

A hands-on project to model, configure and integrate a real laboratory workflow in an Electronic Lab Notebook (ELN), from requirements to data integration.

**Status:** In progress (started October 2026)
**Type:** Personal learning project – all data is fictional

---

## Why this project

I am a PhD-trained life scientist with several years of experience supporting automated protein analysis platforms in biopharma and academic labs. I know the bench side of lab workflows well; this project is about the digital side: how a workflow is structured, captured and connected inside lab informatics systems.

The goal is to go through the full cycle a Solution Consultant or Business Analyst would follow on an ELN project:

1. Understand the scientific workflow and the users' needs
2. Model the data (samples, reagents, instruments, results)
3. Configure the ELN (resources, templates, statuses)
4. Integrate instrument data through an API
5. Transpose the data model to an enterprise platform (Benchling-style)
6. Document requirements, user stories and acceptance testing

## The scenario

A fictional biotech characterizes antibody candidates using **Simple Western** (capillary-based automated Western blot). Today, results are scattered across instrument exports, Excel files and slide decks. Nobody can easily answer: *"Which experiments used antibody lot X, and what were the results?"*

This project builds the structure that makes that question answerable.

## Tools

| Tool | Use |
|---|---|
| [eLabFTW](https://www.elabftw.net) | Open-source ELN: experiments, resources, templates, audit trail |
| Docker Compose | Running eLabFTW, MySQL and the reverse proxy as three containers |
| [Caddy](https://caddyserver.com) | HTTPS reverse proxy in front of eLabFTW |
| Python + eLabFTW REST API v2 | Importing instrument results into experiments *(planned)* |
| [Benchling documentation](https://help.benchling.com) | Reference for the enterprise data model (registry, inventory, result schemas) |
| draw.io | Architecture and data model diagrams *(planned)* |

## Roadmap

- [x] Explore eLabFTW: experiments vs. resources, linking, revision history
- [x] Self-host eLabFTW 6 locally (Docker Compose, MySQL, Caddy HTTPS reverse proxy)
- [ ] Set up the team, users and roles
- [ ] Model the Simple Western workflow: resource types, experiment template, statuses
- [ ] Data integrity review: audit trail and revisions mapped to ALCOA+ principles
- [ ] Python integration: CSV instrument export → validation → eLabFTW via API
- [ ] Solution architecture diagram (including error handling and a Benchling variant)
- [ ] Benchling-style data model for antibody characterization
- [ ] Business analysis deliverables: discovery guide, user stories, UAT plan, build-vs-buy note
- [ ] Demo video and final write-up

## Repository structure *(will grow as the project progresses)*

```
.
├── README.md            ← overview and learning log
├── 00-infrastructure/   ← docker-compose.yml and Caddyfile (no secrets)
├── 01-exploration/      ← notes and screenshots from the first sessions
├── 02-data-model/       ← resource types, experiment template, statuses
├── 03-integration/      ← Python script, sample data, architecture diagram
├── 04-benchling-model/  ← data model design note and diagram
└── 05-business-analysis/← discovery guide, user stories, UAT plan
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
- **Test vs. production.** This local setup uses a self-signed certificate and fictional data. A production deployment (Linux server, certificate from a trusted authority, backups, monitoring) is a different scope.

---

## Notes

- **All data in this project is fictional.** No real customer, company or patient data is used.
- This is an independent learning project. It is not affiliated with, endorsed by or developed for any company or software vendor mentioned here.
- Some parts were drafted with AI assistance; all content was reviewed, tested and validated by me.
