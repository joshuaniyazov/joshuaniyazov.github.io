# Joshua Niyazov

## CS-499 Computer Science Capstone ePortfolio

Welcome to my Computer Science Capstone ePortfolio. This site showcases my work in three key areas of computer science:

- **Software Design and Engineering**
- **Algorithms and Data Structures**
- **Databases**

All three categories are represented through enhancements to a single full-stack project:

> **LinkHub** – a secure, customizable link-in-bio platform built with Node.js, Express, MySQL, and EJS.

---

## Table of Contents

1. [Professional Self-Assessment](#professional-self-assessment)  
2. [Code Review](#code-review)  
3. [Artifact One: Software Design and Engineering](#artifact-one-software-design-and-engineering)  
4. [Artifact Two: Algorithms and Data Structures](#artifact-two-algorithms-and-data-structures)  
5. [Artifact Three: Databases](#artifact-three-databases)  
6. [Course Outcomes Mapping](#course-outcomes-mapping)  
7. [Repository Links](#repository-links)  
8. [Contact](#contact)

---

## Professional Self-Assessment

Throughout my Computer Science program, I have had the opportunity to deepen my technical knowledge, strengthen my problem-solving abilities, and refine the professional habits that will guide my career. Completing my coursework and developing this ePortfolio has helped me recognize the full breadth of my strengths and translate them into tangible demonstrations of competence. This process pushed me to revisit earlier projects, elevate them with modern engineering practices, and clearly articulate the reasoning behind my design choices. As a result, this portfolio showcases not only what I can build, but also how I think, how I collaborate, and how I approach complex problems.

Across the program, I completed a wide range of projects that shaped my skill set. I worked with languages and technologies such as Python, Java, C++, JavaScript, Node.js, SQL, and MySQL. Courses like data structures and algorithms strengthened my understanding of runtime trade-offs and data organization, while software engineering and database courses gave me hands-on experience designing full-stack systems, drawing UML diagrams, writing technical documentation, and implementing CRUD operations in relational databases. My capstone work on LinkHub builds directly on these experiences by turning a functional prototype into a more secure, modular, and scalable system.

The program also emphasized collaboration and communication. In team-based courses, I practiced version control, code reviews, and change management, which taught me how to work productively with other developers. Communicating with stakeholders—whether real or simulated—helped me learn how to translate non-technical requirements into actionable technical plans. These experiences have been invaluable in preparing me for professional environments where teamwork and clear communication are just as crucial as technical skill.

From a technical standpoint, I developed strength in **data structures and algorithms**, **software engineering**, **databases**, and **security**. I learned to reason about algorithmic efficiency, design modular architectures, write maintainable code, and design schemas that protect data integrity. Just as importantly, I learned to adopt a **security mindset**—anticipating adversarial behavior, validating inputs, and treating every interface as a potential attack surface. These principles are reflected in the enhancements I made to LinkHub, where I improved file handling, tightened access control, and strengthened database design.

Collectively, these experiences have shaped my professional identity as a developer who values clarity, correctness, security, and thoughtful design. I am now better prepared to contribute as a software engineer, especially in backend and full-stack roles that demand both algorithmic thinking and robust engineering practices.

---

## Code Review

As part of the capstone, I performed a comprehensive **code review** of the original LinkHub project. This review covered:

- Existing functionality (Express routing, EJS templating, MySQL integration)
- Software architecture and design (monolithic `server.cjs` file)
- Algorithms and data structures (initialization logic, settings map creation, IP parsing)
- Database design (tables, keys, constraints, indexes)
- Security posture (upload handling, debug endpoints, settings exposure)

The code review identified several key issues:

- A **monolithic architecture** where nearly all logic lived inside `server.cjs`
- An unauthenticated `/debug/settings` route that exposed configuration data
- Deterministic avatar filenames that could collide on upload
- Repeated, inefficient initialization queries and redundant settings map computation
- Missing database indexes and lack of transactions for multi-step admin operations

These findings guided the enhancement plan that I later implemented in three categories.

> 🎥 **Code Review Video:**  
> *TODO: Insert YouTube link to your code review video here.*

> 📊 **Code Review Slides (PDF/PPTX):**  
> *TODO: If you upload your slide deck to GitHub, link to it here.*

---

## Artifact One: Software Design and Engineering

### Artifact Description

**Artifact:** LinkHub – Core Server Architecture (`server.cjs`)  
**Category:** Software Design and Engineering  
**Technology Stack:** Node.js, Express, EJS, MySQL  

LinkHub is a link-in-bio platform that lets an administrator configure links, embeds, and settings that render on a public profile page. In the original implementation, almost all application behavior was contained in a single file, `server.cjs`, which handled:

- Express bootstrapping and middleware
- Helmet Content Security Policy configuration
- Session and CSRF protection
- File uploads
- Admin dashboard and public routes
- CRUD handlers for links, embeds, and settings
- Redirect logic

This monolithic design worked, but it created tight coupling and made the system hard to test, extend, or secure.

### Software Design Issues Identified

- **Single-file architecture:** `server.cjs` (lines 34–389) contained nearly all initialization and logic.
- **Security misconfiguration:** `/debug/settings` (originally lines 413–415) returned all settings without authentication.
- **Unsafe file handling:** Avatar uploads used deterministic filenames like `avatar.png`, allowing collisions and overwrites.

These issues violated key software engineering principles such as separation of concerns, least privilege, and defensive programming.

### Enhancements Implemented

I focused my design/engineering enhancements on three main areas:

1. **Hardened Uploads (Design + Security)**
   - Implemented random hex-based filenames for avatars and OG images to prevent collisions.
   - Added upload validation and safer file handling patterns.
   - Introduced helper logic to delete old uploads only when they are within the intended `/static/uploads` directory, reducing the risk of stale or orphaned files.

2. **Secured Diagnostics**
   - Modified `/debug/settings` to be **admin-only** and **disabled in production**, closing the unauthenticated configuration leak.
   - This aligns with secure coding practices and prevents unprivileged users from reading site configuration.

3. **Architectural Improvements and Modularization Plan**
   - Separated conceptual responsibilities: configuration, database helpers, settings management, routing, and middleware.
   - While the capstone timeframe did not require a full multi-file rewrite, the enhancements prepare the codebase for a modular structure (e.g., `config.js`, `db.service.js`, `settings.service.js`, `routes/`, `middleware/`).

### Why This Artifact Demonstrates Software Engineering Skills

This artifact shows my ability to:

- Analyze a real-world codebase and diagnose architectural and security weaknesses.
- Plan and implement improvements that make the system more maintainable and secure.
- Balance pragmatic concerns (existing structure, time constraints) with ideal design patterns.
- Move a prototype toward a production-grade architecture.

---

## Artifact Two: Algorithms and Data Structures

### Artifact Description

**Artifact:** LinkHub – Secure Upload and File Lifecycle Algorithm  
**Category:** Algorithms and Data Structures  

This artifact focuses on the algorithms behind file uploads and lifecycle management in LinkHub. The goal was to design algorithms that were:

- Collision-resistant (no filename conflicts)
- Safe against path traversal or accidental deletion
- Efficient and maintainable

### Algorithmic Issues Identified

- **Deterministic Filenames:** Using fixed names like `avatar.<ext>` created a race condition when multiple users uploaded files concurrently, allowing overwrites.
- **No Cleanup Logic:** Old uploads were never deleted, leading to stale files accumulating on disk.
- **Naive IP Parsing and Redundancy (broader algorithmic context):**
  - `getSettingsMap` was called multiple times per request.
  - IP parsing trusted unvalidated header fields.

### Enhancements Implemented

1. **Random Hex-Based Filenames**
   - New uploads generate secure, random hex identifiers for filenames.
   - This algorithm drastically reduces the chance of collisions and ensures that each file is uniquely addressable.
   - It also improves compatibility with caching/CDN strategies.

2. **File Lifecycle Cleanup Algorithm**
   - Implemented `deleteUploadIfLocal` helper logic that verifies whether a previous file is:
     - Non-null
     - Within the `/static/uploads` subtree
   - If so, it safely deletes the file when a new one replaces it.
   - This prevents stale files from lingering and avoids accidental deletion of unrelated paths.

3. **Validation and Guardrails**
   - Added MIME-type and path checks before processing or deleting files.
   - Ensured that cleanup operations are deterministic and safe, even in edge cases.

### Why This Artifact Demonstrates Algorithm & Data Structure Skills

This artifact shows my ability to:

- Apply algorithmic principles to solve practical engineering problems (collisions, lifecycle management).
- Design safe, efficient routines that manage state across the filesystem and application.
- Consider edge cases, concurrency, and security when designing algorithms.
- Improve the correctness and maintainability of critical application logic.

---

## Artifact Three: Databases

### Artifact Description

**Artifact:** LinkHub – MySQL Schema and Persistence Layer  
**Category:** Databases  

LinkHub uses a MySQL database to manage users, links, settings, metrics, likes, embeds, and redirects. The database layer work is primarily in:

- Table creation and schema definition
- Prepared statements and pooled query helpers
- Admin CRUD handlers
- Metrics and tracking routines

### Database Issues Identified

- **Missing Indexes**
  - Tables frequently filtered or sorted by `order_index`, `is_visible`, or `slug` lacked supporting indexes.
  - This forced full table scans and harmed scalability.

- **No Transactions for Multi-Step Admin Operations**
  - Routes such as `/admin/settings` and `/admin/link` performed multiple updates (and sometimes filesystem changes) without transactional control.
  - Failures mid-operation could leave the database in an inconsistent state.

- **Weak Constraints**
  - Fields like `site_url` or redirect URLs were validated only in application code.
  - The database would accept malformed URLs or unintended values, weakening data integrity.

### Enhancements Implemented

1. **Indexing and Performance Improvements**
   - Added composite indexes on `(order_index, is_visible)` for links and embeds.
   - Added a covering index on `redirects.slug` to accelerate redirect lookups.
   - These changes reduce query time and help the application scale as the dataset grows.

2. **Transactional Safety**
   - Refactored admin operations that involve multiple statements (and sometimes file changes) to use `BEGIN`/`COMMIT` transactions.
   - Ensures that either all changes succeed or none do, keeping the database consistent with the UI.

3. **Schema Hardening and Constraints**
   - Tightened URL and length constraints at the SQL level (e.g., `CHECK` constraints, stricter column sizes).
   - This enforces data integrity even if an application bug surfaces, allowing issues to be caught earlier.

### Why This Artifact Demonstrates Database Skills

This artifact shows my ability to:

- Analyze and optimize relational schemas for performance and scalability.
- Use indexes effectively to support common query patterns.
- Employ transactions to preserve consistency across complex updates.
- Enforce integrity through schema-level constraints, not just application logic.

---

## Course Outcomes Mapping

Below is a summary of how my work in this capstone and the LinkHub enhancements support the CS program outcomes:

1. **Collaborative Environments**  
   - Conducted a thorough code review as if collaborating with a team, documenting findings and planned enhancements.
   - Applied practices similar to professional code collaboration and decision-making.

2. **Professional Communication**  
   - Produced written narratives, presentations, and this ePortfolio to explain technical decisions clearly and coherently.
   - Structured explanations for different audiences (instructors, peers, potential employers).

3. **Algorithmic Design and Problem Solving**  
   - Designed and implemented the secure upload and file lifecycle algorithms.
   - Optimized initialization routines, settings handling, and data flows using algorithmic principles.

4. **Software Engineering / Databases**  
   - Refactored and hardened the LinkHub architecture, improved uploads, and secured admin endpoints.
   - Enhanced database schema with indexes, transactions, and constraints to deliver real value and scalability.

5. **Security Mindset**  
   - Locked down `/debug/settings`, validated file inputs, and added defensive checks around filesystem operations.
   - Treated the database and file handling logic as potential attack surfaces and mitigated those risks.

---

## Links

- 🔗 **LinkHub Repository (Enhanced Artifact):**  
  https://github.com/Jishwuh/LinkHub

---

## Contact

If you would like to connect or learn more about my work:

- **GitHub:** https://github.com/Jishwuh

Thank you for visiting my CS-499 ePortfolio.
