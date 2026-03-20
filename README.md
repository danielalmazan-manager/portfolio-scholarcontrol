# ScholarControl EMERITUS — Enterprise Architecture Case Study

> **End-to-End SaaS Platform for Graduate Program Operations**
> Designed & delivered by **Daniel Almazán** · IT Project Manager & Technical Architect

---

![PHP](https://img.shields.io/badge/Backend-PHP_8+-777BB4?style=for-the-badge&logo=php&logoColor=white)
![MySQL](https://img.shields.io/badge/Database-MySQL_8-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![JavaScript](https://img.shields.io/badge/Frontend-Vanilla_JS-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Bootstrap](https://img.shields.io/badge/UI-Bootstrap_5-7952B3?style=for-the-badge&logo=bootstrap&logoColor=white)
![Chart.js](https://img.shields.io/badge/Analytics-Chart.js-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)
![PHPMailer](https://img.shields.io/badge/Notifications-PHPMailer_SMTP-00A651?style=for-the-badge&logo=gmail&logoColor=white)
![SheetJS](https://img.shields.io/badge/Data_Import-SheetJS-1D6F42?style=for-the-badge&logo=microsoftexcel&logoColor=white)
![Composer](https://img.shields.io/badge/Deps-Composer-885630?style=for-the-badge&logo=composer&logoColor=white)

---

> **⚠️ Confidentiality Notice**
> This repository is a **public architecture case study only**. Per NDA agreements with the client organization, no proprietary source code, credentials, student data, or internal business logic is included. All references describe system capabilities at a design level to demonstrate architectural thinking, systems integration skills, and end-to-end project delivery.

---

## Table of Contents

- [The Business Challenge](#the-business-challenge)
- [My Role & Scope of Ownership](#my-role--scope-of-ownership)
- [Solution Architecture](#solution-architecture)
- [Technology Stack](#technology-stack)
- [Module Breakdown](#module-breakdown)
- [Systems Integration & Automation](#systems-integration--automation)
- [Security Architecture](#security-architecture)
- [Impact & Measurable Results](#impact--measurable-results)
- [Architecture Diagram](#architecture-diagram)
- [Lessons Learned](#lessons-learned)
- [Contact](#contact)

---

## The Business Challenge

The client — a regional operations team managing graduate-level programs for a global EdTech institution — faced compounding operational bottlenecks:

- **Fragmented tooling.** Student records, cohort assignments, course run tracking, and partner communications lived across disconnected spreadsheets, shared drives, and limited SaaS tools (Airtable, Google Sheets). No single source of truth existed.
- **Manual back-office overhead.** Admission teams spent 60–70% of their time on repetitive data entry: enrolling students into cohorts, tracking status changes, and manually emailing institutional partner contacts when a student's enrollment status changed (drops, reactivations, transfers).
- **Zero visibility for leadership.** Program managers and coordinators had no real-time dashboard or consolidated KPI view. Reporting required ad-hoc spreadsheet assembly, introducing delays and data-quality risks.
- **Scalability ceiling.** As the program catalog grew (new courses, new cohorts per quarter, new institutional partners), the manual processes could not scale without proportionally increasing headcount.
- **Audit trail gaps.** There was no structured history of student status changes, no traceability of who made a change, and no automated notification chain when critical events occurred.

The organization needed a **purpose-built, centralized platform** that could unify operations, enforce data integrity, automate repetitive workflows, and provide executive-level visibility — all without the licensing cost and rigidity of a commercial CRM.

---

## My Role & Scope of Ownership

**Title:** IT Project Manager & Technical Architect

I owned this initiative **end-to-end** — from initial requirements gathering and stakeholder alignment through database schema design, full-stack development, deployment, and post-launch iteration.

| Responsibility | Details |
|---|---|
| **Requirements Engineering** | Conducted discovery sessions with Admission, Academic Coordination, and Partner Relations teams to map existing workflows and pain points |
| **System Architecture Design** | Designed the modular monolith architecture, database schema (15+ tables), API contract layer, and role-based access model |
| **Full-Stack Development** | Built backend APIs (PHP 8 / MySQLi), frontend shell (JS / Bootstrap 5), and all seven functional modules |
| **Data Migration Strategy** | Designed and executed the bulk import pipeline (Excel/CSV → validation → upsert) to onboard historical student records |
| **DevOps & Deployment** | Configured Apache/Nginx server blocks, Composer dependency management, SSL, and SMTP relay integration |
| **Stakeholder Communication** | Delivered sprint demos, managed feedback loops, and documented the system for future maintainers |

---

## Solution Architecture

The platform follows a **modular monolith** pattern — a single deployable PHP application composed of self-contained page modules, each with its own API endpoint, view layer, and client-side logic. This architecture was chosen deliberately over a microservices approach for three reasons: (1) team size (solo developer), (2) deployment simplicity on shared hosting infrastructure, and (3) speed-to-market.

### Core Architectural Decisions

- **Dynamic SPA-like Shell.** The main `index.php` acts as an application shell. A persistent sidebar and navbar remain fixed while the content area loads page modules asynchronously via `fetch()`. Each module is a self-contained `pages/{module}/` directory containing `view.php`, `script.js`, and `style.css`. This delivers SPA-like navigation without a framework dependency.
- **Convention-based Module System.** Adding a new module requires only three steps: create the directory, register the sidebar link, and the app router (`app.js`) handles asset loading, cleanup, and hash-based navigation automatically. This was documented in an internal `NEW_PAGE_GUIDE.md` for team handoff.
- **API-per-Module Pattern.** Each module exposes its own `api.php` endpoint using a `switch($action)` dispatcher. This keeps module logic isolated while sharing the global database connection and authentication context.
- **Pluggable Mail Engine.** The notification subsystem abstracts email dispatch behind a configurable `MAIL_METHOD` switch (`smtp` | `mail` | `none`), enabling zero-change toggling between PHPMailer SMTP, native `mail()`, and a dry-run mode for local development.

---

## Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Backend** | PHP 8+ (Native) | API endpoints, server-side auth, CSRF protection, business logic |
| **Database** | MySQL 8 / MariaDB 10.5+ | Relational data store with `utf8mb4` encoding, prepared statements throughout |
| **Frontend** | Vanilla JavaScript (ES6+) | Async module loading, DOM manipulation, form handling, file parsing |
| **UI Framework** | Bootstrap 5 | Responsive layout, modal system, form components |
| **Data Visualization** | Chart.js | Dashboard doughnut charts (student demographics, course run distribution) |
| **Rich Dropdowns** | Choices.js | Searchable select inputs for program codes, cohort assignments, partner contacts |
| **File Parsing** | SheetJS (XLSX) | Client-side Excel/CSV parsing for the student bulk import pipeline |
| **Email Dispatch** | PHPMailer 7 via Composer | SMTP relay integration (Brevo/SendinBlue) for automated partner notifications |
| **Package Management** | Composer | PHP dependency management (autoloading, PHPMailer) |
| **Security** | CSRF tokens, `password_hash()`, prepared statements, session hardening | Defense-in-depth across all layers |

---

## Module Breakdown

The system comprises **seven functional modules**, each serving a distinct operational domain:

### 1. Executive Dashboard
Real-time command center with glassmorphism-styled KPI cards. Displays total students, active course runs, cohort count, average NPS score, and average completion rate — all animated on load. Two Chart.js doughnut visualizations show student status distribution and course run status breakdown. Bottom tables surface the five most recent course runs and student enrollments.

### 2. Program & Catalog Management
Three-tab interface for managing the foundational data hierarchy: **Programs** (parent course codes), **Products** (program categories), and **Partners** (institutional affiliates). Programs expand inline to reveal their associated courses. Column-level dropdown filters enable rapid data navigation across large catalogs.

### 3. Course Runs (Running Courses)
Tracks individual course instances with start/end dates, assigned Primary and Secondary ADMs, run status, completion rate, NPS evaluation, and course rating. Features a **cohort linking subsystem** — course runs are associated with one or more cohorts via a junction table (`TableRunCohort`), with an in-modal interface for linking/unlinking batches with searchable dropdowns.

### 4. Cohort & Student Management
The operational core of the platform. A master-detail interface where selecting a cohort reveals its student roster. Key capabilities include:
- **Individual student CRUD** with full status tracking
- **Bulk import pipeline**: Upload CSV/XLSX → client-side SheetJS parsing → server-side duplicate validation → editable preview grid → transactional upsert with automatic history logging
- **Status history system**: Every status change creates an auditable record in `TableHistoryStatusStudentList`, capturing the user who made the change, timestamps, reactivation dates, and new batch assignments
- **Partner contact association**: Status changes can be linked to one or more partner contacts, with a **one-click email notification** workflow that dispatches formatted HTML emails via the SMTP relay
- **Mass status change**: Checkbox selection across students enables bulk status transitions with a single form submission
- **Quick-add partner contact**: Inline form within the history modal allows creating new partner contacts without leaving the workflow

### 5. Course Grades
Grading interface supporting two evaluation methods: numerical rating (0–100) and status-based rating (Pass/Fail). Features a **batch grading utility** that applies a selected method and value to all checked students simultaneously, with a cascading highlight animation for visual feedback. Grades are persisted per student per course run via `TableHighRatings`.

### 6. Partner Contacts
Full CRUD management for institutional partner contacts with activate/deactivate toggles, duplicate email validation, and column-level filtering by partner and status. Contacts created here are available system-wide for association with student status changes and automated notifications.

### 7. User Access Control
Administrator-only module for managing system accounts. Supports user creation with `password_hash()` encryption, role assignment via the `TableRol` catalog, and active/blocked status toggling with self-blocking prevention. Column filters for role and status enable quick auditing.

---

## Systems Integration & Automation

### Automated Partner Notification Pipeline

One of the highest-value automations in the system:

```
Student Status Change → History Record Created → Partner Contacts Associated
        ↓
   User clicks "Save & Notify"
        ↓
   System queries joined data (student info + status + reason + contacts)
        ↓
   PHPMailer dispatches formatted HTML emails via SMTP relay
        ↓
   UI reports success/failure count per recipient
```

This replaced a **fully manual process** where admission team members had to (1) update a spreadsheet, (2) compose an email, (3) look up the correct partner contact, and (4) send individually — per student, per status change.

### Excel/CSV Bulk Import Pipeline

```
User uploads .xlsx/.csv → SheetJS parses client-side → Date normalization
        ↓
   Parsed rows sent to server for duplicate validation
        ↓
   Editable preview grid with per-row status/batch overrides
        ↓
   Confirmed records inserted via transactional upsert
        ↓
   Initial history records auto-generated per student
```

Supports multiple date formats (ISO, DD/MM/YYYY, Excel serial numbers), handles duplicate detection against existing database records, and allows reassignment of duplicate students to different cohorts before import.

---

## Security Architecture

Security was designed as a **defense-in-depth** strategy, not an afterthought:

| Concern | Implementation |
|---|---|
| **Authentication** | Session-based login with `password_hash()` / `password_verify()` (bcrypt) |
| **Session Hardening** | `httponly`, `SameSite=Strict` cookie flags; HTTPS-ready configuration |
| **CSRF Protection** | Per-session token generated via `random_bytes(32)`, validated on every non-GET request via custom `X-CSRF-Token` header; global interceptor in `app.js` auto-injects the token |
| **SQL Injection** | 100% prepared statements with parameterized queries (`bind_param`) across all 7 API modules |
| **XSS Prevention** | `htmlspecialchars()` on all user-facing output in notification templates |
| **Authorization** | Role-based route guarding at both PHP (API-level) and sidebar (UI-level); roles 2, 3, and 5 have differentiated access |
| **Path Traversal** | Whitelist-based page loading — `allowedPages` array in `app.js` prevents LFI attacks via hash manipulation |
| **Credential Isolation** | Database credentials and SMTP keys isolated in `config.php`, excluded from version control via `.gitignore` |

---

## Impact & Measurable Results

| KPI | Before | After |
|---|---|---|
| **Student onboarding time** | 15–20 min/student (manual entry) | < 2 min/batch (bulk import for 50+ students) |
| **Partner notification turnaround** | 24–48 hours (manual email composition) | < 30 seconds (one-click automated dispatch) |
| **Data source consolidation** | 5+ disconnected tools | 1 centralized platform |
| **Status change audit trail** | Non-existent | 100% traceable (user, timestamp, reason, contacts notified) |
| **Executive reporting setup** | Hours of spreadsheet assembly | Real-time dashboard with live KPIs |
| **Time spent on repetitive admin tasks** | ~60–70% of admission team workweek | Reduced by an estimated **50%+** |
| **Platform scalability** | Spreadsheet row limits | Designed for thousands of students across unlimited cohorts and course runs |

---

## Architecture Diagram

> `[Insert Architecture Diagram Here]`
>
> *A high-level block diagram illustrating the modular monolith structure, database relationships, the dynamic page-loading shell, SMTP integration layer, and role-based access flow will be placed here.*

---

## Lessons Learned

1. **Convention over configuration scales solo projects.** The `pages/{module}/` convention with automatic asset loading meant I could ship new modules in hours, not days. Documenting the pattern (`NEW_PAGE_GUIDE.md`) ensured the system remained extensible after handoff.

2. **Client-side validation is UX; server-side validation is security.** The bulk import pipeline validates on both sides — SheetJS parsing catches format issues instantly for the user, while the server-side duplicate check enforces data integrity. Neither alone is sufficient.

3. **Abstracting the mail layer was worth the upfront cost.** The `MAIL_METHOD` switch (`smtp` / `mail` / `none`) saved significant debugging time during development and made deployment to environments with different mail infrastructure seamless.

4. **Role-based access needs to be enforced at every layer.** Hiding a sidebar link is not security. Every API endpoint independently validates the session role before executing, ensuring that direct API calls cannot bypass UI restrictions.

5. **A modular monolith is a legitimate architecture.** Not every project needs microservices. For a solo-developed internal tool with a clear deployment target, the modular monolith delivered fast iteration, simple debugging, and zero operational overhead from service orchestration.

---

## Contact

**Daniel Almazán**
IT Project Manager · Technical Architect · Systems Integration Specialist

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/YOUR-PROFILE)
[![Email](https://img.shields.io/badge/Email-Contact_Me-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:your.email@domain.com)
[![Portfolio](https://img.shields.io/badge/Portfolio-View_More-000000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/YOUR-USERNAME)

---

<sub>This case study was authored for portfolio purposes. No proprietary code, credentials, or protected data is included in this repository. All technical descriptions reflect architectural decisions and system capabilities at a design level.</sub>
