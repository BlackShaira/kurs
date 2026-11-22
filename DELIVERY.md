# What ticket #4471 actually holds

This repo is the support desk's knowledge base, as it was exported on 2026-10-04, with one page
added about what `#4471` really holds, and one callout added to the desk's own page for that
ticket so a reader of #4471 reaches the new page. The kb is here so the added page sits in the
base it belongs to, with its imports resolving and its tables built from the desk's own data.

| Path in this repo | What it is |
|---|---|
| `README.md`, `src/` | the kb as exported, byte for byte, apart from the one file below |
| `src/routes/kb/incidents/4471/+page.svx` | the desk's own incident page, with one callout added. Nothing in it was removed or rewritten |
| `src/routes/kb/incidents/4471/split/+page.svx` | the page added by this work |
| `src/lib/kb/SplitTable.svelte` | the component that page uses, added by this work |
| `rendered/kb/incidents/4471/split/index.html` | that page as the desk will see it |
| `DELIVERY.md` | this file |

The desk's reports are `src/lib/data/reports-4471.json`, unchanged: every one of them still
carries `mergedInto: 4471`. The ticket a report belongs to is that field and nothing else, so
moving a report would take it out of the desk's view of #4471 altogether.

## The problems in #4471

| Problem | Reports | Ticket |
|---|---|---|
| the parser reads every date month first | 379 | issue #1 of this repo, and `#4471` itself on the kb |
| `booking-v3` asks for `MM/DD/YYYY` and stores the date day-first | 33 | issue #2 of this repo |
| invites land one day out | 5, inside the 379 | none, and it needs none |

Counted the way the desk's own view counts, as it was exported: `groupByCause` in
`src/lib/kb/causes.ts` over `uniqueById` of the reports. The two causes are 379 and 33, which add
up to the 412 the ticket holds. Do not take those numbers from this file — `CauseTable` on
#4471's page and `SplitTable` on the added page both print them from the data at render time,
which is the kb's own rule: a number that can be computed belongs in a component.

## Why the third problem has no ticket

`causes.ts` separates two cases, and only one of them is a ticket's worth.

Reports whose cause is not the ticket's cause were, in its own words, *merged into the wrong
place*. That is the `booking-v3` hint group, and issue #2 is the ticket it never had.

The `oneDayMove` reports in `src/lib/data/desk-notes.json` are the other case: `readAs` counts
them *where they're filed, footnoted under it*. They are in the right ticket and read the wrong
way, so they stay inside the parser count and are footnoted there. The problem is real, and the
added page names it and quotes every one of those reports — but it is not a cause, and the kb
gives it no ticket slot.

## Where the numbers come from

Every count is produced by the knowledge base's own logic, not by a rule invented here:

- `uniqueById` first. The export ships **427 report objects for 412 reports**, because its export
  pages overlap. The duplicate copies are identical.
- `causeOf(report, tenant)` for the two causes. It asks the platform whether the tenant's locale
  writes the day first, and treats a report through a `booking-v3` form as a different cause.
- `readAs` with `desk-notes.json` for the third problem, which is how the desk's own view reports
  it: counted where it is filed, footnoted under it.

Do not count by `meta.intakeTag`. Intake guesses the cause from the reporter's browser the moment
the report arrives, and its guess disagrees with the cause the report belongs to for a sizeable
minority of them, which `SplitTable` prints.

## How the rendered page was produced

The export ships `src/` only: no `package.json`, no build setup, so mdsvex and Svelte cannot run
against it. The HTML in `rendered/` was produced by a renderer that reads the `.svx` source
itself, so the words have one source and the two files cannot drift apart. The frame, the
component markup and the cause logic are copied from the export verbatim where they are logic.
Build the `.svx` in the real knowledge base and the page is the same page; the HTML here is for
reading it without a build.
