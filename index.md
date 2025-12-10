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

I have developed my knowledge, problem-solving, and professional practices throughout the Computer Science curriculum, and these will serve me well on my career path. Completing my coursework and creating this ePortfolio have given me the chance to realize my skills and demonstrate them. This project allowed me to go back and assess previous work, upgrade it using new technology, and discuss why I made particular design choices. This portfolio works not only at demonstrating what I can do, but also how I think, collaborate with other people, and deal with difficult problems.

During the course of the program, I have undertaken numerous projects that helped mold my skills. I have worked with languages and technologies such as Python, Java, C++, JavaScript, Node.js, SQL, and MySQL. Courses undertaken on data structures and algorithms honed my understanding of the trade-offs that happen during runtime and data structuring, and courses on software engineering and database provided me with practicals on the design of full-stack applications, UML diagram creation, the generation of tech docs, and the implementation of CRUD operations for relational databases. My capstone project for LinkHub extends these concepts by improving the prototype into one that is more secure, modular, and scalable.

The curriculum also promoted teamwork and communication. Through the team-based courses, I developed skills in version control, code reviewing, and managing changes, which have taught me the importance of collaboration when working with other programmers. Interacting with the stakeholders, whether it is real or simulated, has helped me understand how to convert the non-technical requirements into effective technical plans. Such skills will help me perform well when I enter the industry since communication and collaboration will matter equally compared to my technical skills.

Technically, I was able to enhance my skills in data structures and algorithms, software engineering, database systems, and security. I learned how to assess the efficiency of algorithms, build robust architectures, write maintainable code, and build schemas that safeguard data. I also developed a security-focused approach, where I learned how to think about malicious users, verify inputs, and view every interface as a vulnerability. I was able to implement these concepts at LinkHub, where I enhanced file management, secured access, and improved the database structure. Taking everything into consideration, I have developed my professional identity as a software developer concerned about clarity, correctness, security, and design. I feel that I will be better equipped for the role of a software engineer, specifically for backend or full-stack development, where sound thinking is essential.

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

## Repository Links

- 🔗 **LinkHub Repository (Enhanced Artifact):**  
  https://github.com/Jishwuh/LinkHub

---

## Contact

If you would like to connect or learn more about my work:

- **GitHub:** https://github.com/Jishwuh
- **SNHU GitHub** https://github.com/joshuaniyazov  

Thank you for visiting my CS-499 ePortfolio.
