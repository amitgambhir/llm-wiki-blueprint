# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## LLM Wiki — SA Knowledge Base

A structured knowledge base for technical leaders, maintained by Claude Code.
Based on Andrej Karpathy's LLM Wiki pattern.

## Purpose

This wiki is a structured, interlinked knowledge base for Solution Architects, Tech Product
Managers, and Tech Program Managers. It compiles architectural decisions, design patterns,
entities, and lessons learned from raw source documents into a persistent, queryable
knowledge graph.

Claude maintains the wiki. The human curates sources, asks questions, and guides the analysis.

---

## Slash Commands

The three primary workflows are exposed as Claude Code slash commands in `.claude/commands/`:

- `/wiki-ingest <path>` — compile a source document from `raw/` into wiki pages (decisions, concepts, entities), update index and log
- `/wiki-query "<question>"` — answer a question from wiki content, optionally file the answer as a synthesis page
- `/wiki-lint` — audit the wiki for orphans, missing links, stale entities, contradictions, and unsourced claims

All three commands read this CLAUDE.md first, then follow the corresponding workflow section below.

---

## Schema Templates

Before creating any new wiki page, read the matching template in `schema/`:

- `schema/decision-template.md`
- `schema/concept-template.md`
- `schema/entity-template.md`
- `schema/synthesis-template.md`

---

## Folder Structure

```text
raw/                    -- source documents (immutable -- never modify these)
  adrs/                 -- Architecture Decision Records
  prds/                 -- Product Requirements Documents
  rfcs/                 -- Design docs and proposals
  transcripts/          -- Meeting notes and call transcripts
  postmortems/          -- Incident reviews and retrospectives
  strategies/           -- Roadmaps, strategy docs, program plans
  external/             -- Articles, papers, vendor docs

wiki/                   -- markdown pages maintained by Claude
  index.md              -- table of contents for the entire wiki
  overview.md           -- living synthesis across all sources
  contradictions.md     -- flagged conflicts between sources
  log.md                -- append-only record of all operations
  decisions/            -- compiled ADR summaries and rationale threads
  concepts/             -- patterns, principles, trade-off frameworks
  entities/             -- systems, teams, vendors, people
  syntheses/            -- filed-back query answers
  questions/            -- open knowledge gaps

schema/                 -- page templates (read before creating any page type)
CLAUDE.md               -- this file
```

---

## Core Principle: Compile, Don't Just Index

When a new source arrives, do not just summarize it in isolation.
**Integrate it** — update existing pages, note contradictions, strengthen or challenge
the current synthesis. The wiki should reflect cumulative understanding, not a flat
collection of summaries. Knowledge is compiled once and kept current.

---

## Ingest Workflow

When the user adds a new source to `raw/` and asks you to ingest it:

1. Read the full source document
2. Discuss key takeaways with the user before writing anything — confirm emphasis and scope
3. Identify all **entities** mentioned (systems, teams, vendors, people) → create or update pages in `wiki/entities/`
4. Identify all **decisions** made → create or update pages in `wiki/decisions/` with full rationale
5. Identify all **concepts** introduced or applied → create or update pages in `wiki/concepts/`, adding this source as evidence
6. Check for **contradictions** → compare new claims against existing wiki pages; flag any conflicts in `wiki/contradictions.md` with both sides cited
7. Add `[[wiki-links]]` to connect related pages throughout
8. Update `wiki/index.md` with new or changed pages and one-line descriptions
9. Update `wiki/overview.md` to reflect the new synthesis
10. Append an entry to `wiki/log.md` with the date, source name, pages created or updated, and contradictions found

A single source may touch 10–20 wiki pages. That is normal and expected.

---

## Page Types and Format

Every wiki page follows this structure:

```markdown
# Page Title

> One sentence describing what this page is about.

**Type**: decision | concept | entity | synthesis | question
**Sources**: list of raw source files this page draws from
**Last updated**: YYYY-MM-DD

---

Main content goes here. Use clear headings and short paragraphs.
Link to related pages using [[wiki-links]] throughout the text.

---

## Related Pages

- [[related-page-1]]
- [[related-page-2]]
```

### Page type: `decisions/`

What was decided, why, what options were rejected and why, trade-offs explicitly accepted,
who decided, when, and consequences. Include a contradiction log at the bottom where later
sources can challenge the original reasoning.
Status field required: `active | superseded | deprecated`

### Page type: `concepts/`

Plain-language definition. When to use and when NOT to use. A trade-offs section (always
present). Evidence listing every decision in this wiki that applies this concept.

### Page type: `entities/`

Role and responsibility. Known strengths and pain points, each backed by a source citation.
All decisions that involve this entity. Include a staleness marker if no source has
referenced this entity in 90 days: `🕐 STALE: last referenced YYYY-MM-DD`

### Page type: `syntheses/`

The question that was asked. The full answer with citations to specific wiki pages.
Tag as `#query-result`. These pages are first-class wiki content — future queries can
draw on them.

### Page type: `questions/`

The unanswered question, why it matters, and what sources might fill the gap.
Status field: `open | in-progress | resolved` (link to synthesis page if resolved)

---

## Citation Rules

- Every factual claim should reference its source using the format: `(source: filename.md)`
- Link to wiki pages using `[[page-name]]` — these become graph edges in Obsidian
- If two sources disagree, note the contradiction explicitly with `⚠️ CONTRADICTION:` followed by both claims and their sources
- If a claim has no source, mark it: `[source needed]`
- Decision pages must link back to the raw source file they were compiled from

---

## Question Answering

When the user asks a question:

1. Read `wiki/index.md` first to identify relevant pages from the one-line descriptions
2. Read only those pages — do not load everything
3. Synthesize an answer and cite specific wiki pages in your response
4. If the answer is not in the wiki, say so clearly — do not speculate
5. If the answer is valuable, offer to save it as a new page in `wiki/syntheses/`

Good answers should be filed back into the wiki so they compound over time.

---

## Lint

When the user asks you to lint or audit the wiki, check for:

- Contradictions between pages
- Orphan pages with no inbound links from other pages
- Concepts mentioned in three or more sources that lack their own page
- Decision pages with no linked concept or entity pages
- Entity pages with the staleness marker that haven't been reviewed
- Claims flagged as `[source needed]`
- Unresolved entries in `contradictions.md` older than 30 days
- Open questions in `questions/` that could now be answered from existing wiki content

Report findings as a numbered list with a suggested fix for each.
Save the report as `wiki/lint-YYYY-MM-DD.md` and append a summary to `wiki/log.md`.

Run lint after every 10 ingests, or whenever the wiki feels inconsistent.

---

## Interlinking Rules

- Every page must link to at least one other wiki page
- Decision pages must link to: the raw source, at least one concept, at least one entity
- Concept pages must link to: at least two decisions that apply them (once they exist)
- When referencing a concept inline, always link it: `[[event-driven-architecture]]`
- Hub pages (`index.md`, `overview.md`) should be the most heavily linked pages in the vault

---

## Writing Conventions

- Keep page names lowercase with hyphens: `api-gateway-decision.md`
- Write in clear, plain language — avoid jargon without definition
- Dates in ISO format: `2026-04-13`
- Decision status always visible: `status: active | superseded | deprecated`
- First line of every page is the one-sentence TLDR prefixed with `>`

---

## Obsidian Notes

This vault is designed to be opened in Obsidian. The Graph View renders every `[[wiki-link]]`
as an edge — hub pages will appear as large nodes. Use Obsidian tags (`#decision`, `#concept`,
`#entity`, `#query-result`, `#open-question`) for filtering in search and graph view.

An `/inbox` folder can be used for rough notes. Ask Claude to triage them:
*"Look at my inbox folder and suggest where each note should be filed and what tags it should get."*

---

## Responsibilities

| Human | Claude |
| --- | --- |
| Drop source files into `raw/` | Read and compile sources into the wiki |
| Confirm emphasis and scope at ingest | Never modify anything in `raw/` |
| Ask questions via natural language | Navigate index, synthesize, file answers back |
| Review wiki pages and resolve contradictions | Maintain cross-links, staleness markers, log |
| Trigger lint periodically | Run lint, produce report, flag gaps |
| Decide what to keep, merge, or deprecate | Propose merges, flag orphans, suggest new pages |

---

## Rules

- Never modify anything in the `raw/` folder
- Always update `wiki/index.md` and `wiki/log.md` after any change to the wiki
- When uncertain about how to categorize something, ask the user before writing
- Do not speculate in wiki pages — if something lacks a source, mark it clearly
- Prefer updating existing pages over creating new ones when content overlaps
