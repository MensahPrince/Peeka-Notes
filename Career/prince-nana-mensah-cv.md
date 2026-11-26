---
id: 3bmwmi9vw8
title: "Prince Nana Mensah CV"
sort: 1000
version: 1
---

**Software Engineer: Systems, Backend and Healthcare Software**

Accra, Ghana · +233 50 026 0063 · +233 53 046 7958 · mensahp5888@gmail.com

Portfolio: [codemesh3d.vercel.app](https://codemesh3d.vercel.app/) · GitHub: \[add your GitHub link\]

## Summary

Software engineer who spent a year building software and running IT support at Korle-Bu Teaching Hospital, Ghana's largest referral hospital. Works mostly in Rust, Go and Python. Has built FHIR-based hospital systems in Django and Docker, desktop apps in Rust with Slint and Iced, and a bare-metal x86\_64 kernel. His largest project is Peeka, a local-first workspace app like Notion, which runs on a block-editor engine he also wrote. A management system he built now runs a physiotherapy clinic's daily operations. Looking for a software engineering role in backend, systems or health tech.

## Technical skills

- **Languages:** Rust, Go, Python, SQL, RISC-V assembly
- **Backend and web:** REST API design, React, Next.js, MySQL, Redis, GoFiber, Actix Web
- **Desktop and systems:** Slint, Iced, Electron, Tauri; CRDTs (Loro); bare-metal x86\_64 (paging, interrupts, IDT), QEMU
- **Infrastructure and tools:** Docker, Linux (Fedora, Ubuntu), Git, pnpm, Postman
- **Healthcare standards:** HL7 FHIR
- **3D and graphics:** Blender, non-photorealistic (NPR) rendering

## Experience

**IT Officer, Korle-Bu Teaching Hospital, Accra** · Sep 2025 to Sep 2026

- Built and maintained software for hospital departments and private healthcare clients, from gathering requirements with clinical staff to deployment.
- Handled day-to-day IT support across \[number\] departments, fixing hardware, network and application problems for clinical and admin staff.
- Led troubleshooting on recurring system faults, cutting repeat tickets by \[X%\] / resolution time by \[X hours\].
- \[Add one specific win, e.g. a system you rolled out, how many users it serves, or a process you automated.\]

**IT Intern, GLICO Group Ltd., Accra** · Sep 2024 to Oct 2024

- Gave first-line IT support and diagnosed problems with staff computers across the office.
- Shadowed the software team's deployments, from build to release.

## Selected projects

**Healthcare software**

- **Hospital Management System** (Django, Docker, HL7 FHIR): full-stack hospital system that exchanges data over FHIR, packaged in Docker so it deploys the same way each time.
- **CITI Physiotherapy Clinic System** (Django): management system that now runs the clinic's day-to-day operations in Ghana.
- **ANKH, Health Information Management System:** designed with clinicians around the way they already work.

**Systems programming**

- **Velora\_OS** (Rust, x86\_64): bare-metal OS kernel with paging, CPU exception handling, hardware interrupts and its own test framework, run under QEMU. *In progress.*
- **Vellum** (Rust): rendering engine that turns 3D scenes into a hand-drawn, anime-style 2D look.
- **GMV, Git Minimal Version** (Rust): version control system that reimplements Git's object model and core commands.

**Desktop and productivity**

- **Peeka** (Rust, Slint): local-first app for pages, notes and databases, like Notion and Obsidian, rewritten from Electron in native Rust. Its databases have table, board, gallery, chart and calendar views, events that span several days, filters such as "due this week" that follow the date, and a week view where events drag to a new time. It has repeating reminders and a schedule view, pages shared and edited live with other people, sync over WebDAV, OneDrive and GitHub, imports from Obsidian and Notion, and export to Markdown and PDF. An MCP server lets AI assistants such as Claude read and edit the pages, databases and reminders. Ships with its own installer for Linux and Windows.
- **Blockwright** (Rust): the editor engine Peeka runs on, split into 19 library crates (about 90,000 lines and 1,000 tests). It handles the document model and undo, Markdown import and export (fuzz-tested), text layout with IME input, TeX math, 14 kinds of chart plus Mermaid diagrams, a database engine with formulas, relations and rollups, and live multi-user editing on Loro CRDTs.
- **Saltr** (Rust, Iced): fast desktop password generator with a simple interface.

**Data and backend tooling**

- **Link-Intel** (Go): API that crawls a site, scores each link from 0 to 1 on how likely it is to lead to downloadable media, and downloads it.
- **Jasynthex** (Go): library that parses large JSON datasets and pulls out key-value paths.
- **YouScrape:** deployed API that extracts structured data from YouTube.

## Education

**BSc Computer Science, Ghana Communication Technology University, Tesano, Accra** · 2021 to 2025

\[Optional: class/CGPA, final-year project title\]

**WASSCE (Biological Science), Saint John's School, Sekondi-Takoradi** · 2017 to 2020

## Editing notes (delete before sending)

The CV now leads with a year of hospital experience instead of asking for an internship, and fits on two pages.

- **Changed:** Summary rewritten in third person. Skills grouped, with Python, Go, Docker and FHIR added (Python was in your summary but missing from skills; Go was listed as "Golang"). Soft-skills section removed, since recruiters skip it and your projects show teamwork better. Experience moved above projects. Projects cut from 19 to 11 and grouped by theme.
- **Removed as too vague or duplicated:** Responsive Web Development, Rust CLI Applications, Django Web Applications, Download Manager UI, PHP client app, Hudly, 3D Modeling (now a skill line).
- **Fill in:** your exact job title at Korle-Bu, numbers in the bracketed bullets, your GitHub link, and CGPA if it's strong.
- **Check:** is Jasynthex the same library as jsonpathfinder? If so, use one name and mention it's published.
- **Consider adding:** REMI, your satellite-imagery system for detecting galamsey and deforestation, once it has a working prototype.
- **Before sending:** export to PDF, name it Prince\_Mensah\_CV.pdf, and drop the P.O. Box; a city is enough.
