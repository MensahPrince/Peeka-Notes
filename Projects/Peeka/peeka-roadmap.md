---
id: q31hpcweeb
title: "Peeka Roadmap"
sort: 2000
version: 1
---

<callout background="blue" icon="📌">

**What Peeka takes from Notion, Obsidian, Logseq, AFFiNE and Anytype**, apart from plugins ([Plugin System Plan](page:r5af9h6xue)). Milestones are in `ROADMAP.md` in the repo root. Next up: **R1 Import**, then **R3 Templates**, then **R2 Page history**.

</callout>

<database-view db="3q3rvio9dl"/>

<database-chart by="wl1am9gyv8" chart="donut" colors="color" db="3q3rvio9dl"/>

## Must have

<toggle>

**R1. Import from Obsidian and Notion**: anyone coming to Peeka comes from somewhere, so this is the door

- [ ] Obsidian vault: `[[links]]`, `![[embeds]]` and `[[Page#Heading]]` converted, frontmatter kept, attachments into assets
- [ ] Notion export zip: nesting, Notion's ids stripped from names, links kept, each CSV as a database
- [ ] A report of what came in and what was left out

</toggle>

<toggle>

**R2. Page history**: losing a page's text is what people fear most

- [ ] Snapshots of personal pages, thinned by age
- [ ] Shared pages: show the history Loro already keeps
- [ ] Browse versions, view one read-only, restore

</toggle>

<toggle>

**R3. Templates**

- [ ] A templates folder, and "New from template" in the sidebar and slash menu
- [ ] `{{date}}`, `{{time}}`, `{{title}}`
- [ ] Per-database row templates

</toggle>

<toggle>

**R4. Relations and rollups**: the core of Notion's databases

- [ ] Two-way relation column
- [ ] Rollups: count, sum, average, list
- [ ] Usable in formulas, in the MCP tools and in import. Waits for S10 so it's built once.

</toggle>

<toggle>

**R5. Export**: being able to leave is a reason to stay

- [ ] Page, folder or workspace as Markdown plus assets, databases as CSV
- [ ] A page as PDF

</toggle>

## What sets Peeka apart

<toggle>

**R6. Daily notes**: today's note from a template, with the agenda and reminders in it

</toggle>

<toggle>

**R7. Quick capture**: a system-wide hotkey opens a small window, and the text goes to the Inbox or today's note

</toggle>

<toggle>

**R8. Synced blocks and embeds**: a block of another page shown and edited in place

</toggle>

<toggle>

**R9. Page properties and tags**: properties on any page, `#tags`, and views over them

</toggle>

## Later

- **R10 Canvas**, **R11 Publishing to the web**, **R12 Web clipper**, **R13 Mobile**, the largest of these. Shared spaces are how a phone would sync.

<callout background="yellow" icon="⚠️">

**Risks:** import's long tail (Dataview and Templater syntax come in as plain text), history's size on disk, and synced blocks crossing between personal and shared spaces.

</callout>
