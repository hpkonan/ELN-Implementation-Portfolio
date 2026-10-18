# ELN Implementation Portfolio – Simple Western Workflow

A hands-on project to model, configure and integrate a real laboratory workflow in an Electronic Lab Notebook (ELN), from requirements to data integration.

**Status:**  In progress (started October 2026)
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
| Python + eLabFTW REST API v2 | Importing instrument results into experiments *(planned)* |
| [Benchling documentation](https://help.benchling.com) | Reference for the enterprise data model (registry, inventory, result schemas) |
| draw.io | Architecture and data model diagrams *(planned)* |

## Roadmap

- [x] Explore eLabFTW: experiments vs. resources, linking, revision history
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

---

## Notes

- **All data in this project is fictional.** No real customer, company or patient data is used.
- This is an independent learning project. It is not affiliated with, endorsed by or developed for any company or software vendor mentioned here.
- Some parts were drafted with AI assistance; all content was reviewed, tested and validated by me.
