# Real-Estate Group Portal

**Case study.** A bilingual corporate platform for a real-estate holding group in the GCC. It combines the public website with a careers portal and applicant matching, a property sales showcase with lead capture, and an investor-relations data pipeline.

> This repository is a write-up, not the source. The code belongs to the client and stays private.
> I'm happy to give a live walkthrough of the architecture and code in an interview.
> Screenshots use an invented brand ("Crescent Bay Holdings") and invented data.

![Homepage](docs/01-home-en.png)

| | |
|---|---|
| **Role** | Sole engineer: full stack |
| **Stack** | React 18 · React Router 6 · react-i18next · Styled Components · Framer Motion · custom SVG charts · Node.js / Express 4 · Sequelize 6 · PostgreSQL · JWT · Multer · SheetJS · node:test |
| **Size** | ~25.8k lines of JavaScript · 19-model schema · 16 route modules / 95 handlers · 12 server tests incl. a Postgres integration test |

---

## The problem

The group's website had to do much more than show brochures. It needed to recruit (job seekers apply with full profiles, and HR needs a ranked shortlist), sell (property listings that turn visitors into inquiries and viewings), and inform investors (financial figures that start life in internal spreadsheets). All of it had to work fully in English and Arabic.

## What I built

### Bilingual public site
- English and Arabic with full right-to-left layout for Arabic, using i18next (EN/AR locale files with identical key sets).

<table><tr><td width="50%"><img src="docs/02-home-ar.png" alt="Arabic RTL"><br><sub>Arabic homepage, full RTL.</sub></td><td width="50%"><img src="docs/10-mobile-home.png" alt="Mobile" width="60%"><br><sub>Mobile layout.</sub></td></tr></table>

- A press-release system and project showcase, both managed from an admin CMS.

### Careers portal and applicant tracking
- Candidate accounts with structured profiles (experience, education, certifications, skills) and document uploads.
- HR admin for vacancies, applications, a searchable CV database and pipeline metrics (status/source counts, average time-to-hire).
- **Weighted candidate matching.** Each job defines mandatory skills, preferred skills, minimum years, location and degree. Candidates are scored with configurable weights and skill synonyms (`js` matches `javascript`, `scrum` matches `project management`). The ranked shortlist can be exported to CSV.

<table><tr><td width="50%"><img src="docs/05-careers.png" alt="Careers"><br><sub>Careers page fed by the jobs API.</sub></td><td width="50%"><img src="docs/06-cv-matching.png" alt="Matching"><br><sub>Candidates ranked by the weighted scorer.</sub></td></tr></table>


### Property sales
- Listings with galleries, text search, and filtering by type, price, location and bedrooms.
- **Lead capture.** Inquiries and appointment bookings are stored per property and managed in the admin.

<table><tr><td width="50%"><img src="docs/03-properties.png" alt="Listings"><br><sub>Listings with filters.</sub></td><td width="50%"><img src="docs/04-property-detail.png" alt="Detail"><br><sub>Detail page with inquiry and booking.</sub></td></tr></table>


### Investor-relations data pipeline
- Finance staff **upload Excel files**. A parser validates each file against **admin-configurable column mappings** (display name, required or optional), so changing a spreadsheet layout is a settings change, not a code change.
- Each upload is **profiled on the way in**: missing required columns, missing values and duplicate rows are counted. Files that fail validation can only be rejected.
- **Two-step maker-checker approval** (analyst, then approver) with a full audit timeline. **Nothing reaches the public investor pages until the final approval**; public APIs read published data only, and caches are invalidated on every decision.
- Dashboards for the income statement, balance sheet, fact sheet and ratios, built from derived metrics with custom SVG charts.

<table><tr><td width="50%"><img src="docs/09-ir-approvals.png" alt="Approvals"><br><sub>Upload awaiting the approver, with validation summary and timeline.</sub></td><td width="50%"><img src="docs/07-ir-income.png" alt="IR dashboard"><br><sub>Public income-statement dashboard (published data only).</sub></td></tr></table>

- An IR document library with bulk upload, published alongside the figures.

---

## Architecture

```mermaid
flowchart LR
    subgraph Client["React SPA (EN / AR, RTL)"]
        PUB[Public site<br/>press · projects · properties]
        JOBS[Careers portal]
        INV[Investors section]
        ADM[Admin CMS]
    end
    subgraph API["Express API"]
        CAR[Careers + matching engine]
        PROP[Properties + inquiries]
        IR[IR upload → validate → approve → publish]
        CMS[Press · projects · docs]
        AUTH[JWT · role guards]
    end
    PUB --> PROP & CMS
    JOBS --> CAR
    INV --> IR
    ADM --> AUTH --> CAR & PROP & IR & CMS
    API --> DB[(PostgreSQL<br/>Sequelize)]
```

### Investor-relations flow

```mermaid
sequenceDiagram
    participant U as Uploader
    participant P as Parser
    participant A as Approver
    participant S as Investor charts
    U->>P: Upload Excel
    P->>P: Map columns · coerce types · count gaps & duplicates
    P-->>U: Reject-only if validation fails
    P->>A: Pending (analyst → approver)
    A->>S: Publish on final approval · cache refresh
    A-->>U: Or reject / request changes with comment
```

## Quality pass (Oct 2026)

A full audit added the first automated tests (unit + a Postgres integration test that drives upload → analyst → approver → public API), gated investor data behind approval, hardened authentication and admin endpoints, fixed crashes in listings, applications and profiles, replaced a mocked job-seeker portal with the real APIs, fixed mobile navigation, and stopped tracking ~66k build/dependency files.

---

*Built by [Fawas Koya](https://github.com/fawaskoya). Client details are anonymized.*
