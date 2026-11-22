# What ticket #4471 actually holds

This repo is the support desk's knowledge base, exactly as it was exported on 2026-10-04,
with one page added to it: the page about what `#4471` really holds. The kb is here so the
added page sits in the base it belongs to, with its imports resolving and its tables built
from the desk's own data, the way the desk would see it.

| Path | What it is |
|---|---|
| `README.md`, `src/` | the kb as exported, unchanged |
| `src/routes/kb/incidents/4471/split/+page.svx` | the page added by this work |
| `src/lib/kb/SplitTable.svelte` | the component that page uses, added by this work |
| `rendered/kb/incidents/4471/split/index.html` | that page as the desk will see it |
| `DELIVERY.md` | this file |

The desk's ticket `#4471` is the kb page `src/routes/kb/incidents/4471/+page.svx`, and its
reports are `src/lib/data/reports-4471.json`. The three tickets this work filed are the issues
of this repo, and each one names that page and that data.

## The three problems

| Problem | Reports | Why it is its own problem |
|---|---|---|
| The parser reads every date month first | 379 | all 11 tenants write the day first, so day and month trade places |
| `booking-v3` asks for `MM/DD/YYYY` and stores the date day-first | 33 | only through the form, only at the 3 tenants on that form; typed month-first on purpose |
| Invites land one day out | 5, inside the 379 | no second number to trade; one meeting reached two timezones on two different days |

Counted the way the desk's own view counts, as it was exported. `groupByCause` splits the
ticket into its two causes, 379 and 33, which add up to 412: every report counted once. The
third problem is the `oneDayMove` list in `desk-notes.json`, five reports the desk read
differently and kept filed under the cause their tenant implies. They are a problem of their
own and need a ticket, and they stay inside the 379, because that is how the desk counts them.

They are filed as the issues in this repo, one per problem, each with its count.

## The page

| File | What it is |
|---|---|
| `src/routes/kb/incidents/4471/split/+page.svx` | the page, in the knowledge base's own format |
| `src/lib/kb/SplitTable.svelte` | the component that counts the three problems at render time |
| `rendered/kb/incidents/4471/split/index.html` | the page as the desk will see it |

The page follows the conventions in the export's own README: words in the page, logic in
`src/lib`, every number computed by a component rather than typed into a paragraph, the
frontmatter fields the layout expects, and a signature with a render stamp at the end.

## Where the numbers come from

Every count is produced by the knowledge base's own logic, in
`src/lib/kb/causes.ts` of the export, not by a rule invented here:

- `uniqueById` first. The export ships **427 report objects for 412 reports**, because its
  export pages overlap. The duplicate copies are identical.
- `causeOf(report, tenant)` for the first two problems. It asks the platform whether the
  tenant's locale writes the day first, and it treats a report through a `booking-v3` form as a
  different cause.
- `desk-notes.json` and `readAs` for the third. Those five ids are the desk's own list, and
  `readAs` is how its view reports them: counted where they are filed, footnoted under it.

Do not count by `meta.intakeTag`. Intake guesses the cause from the reporter's browser the
moment the report arrives, and its guess disagrees with the cause the report belongs to in
**38 of the 412**.

## How the rendered page was produced

The export ships `src/` only: no `package.json`, no build setup, so mdsvex and Svelte cannot
run against it. The HTML in `rendered/` was produced by a renderer that reads the `.svx` source
itself, so the words have one source and the two files cannot drift apart. The frame, the
component markup and the cause logic are copied from the export verbatim where they are logic.
Build the `.svx` in the real knowledge base and the page is the same page; the HTML here is for
reading it without a build.
