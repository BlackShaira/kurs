# What ticket #4471 actually holds

The support desk merged 412 reports about wrong dates into one ticket, `#4471`, and marked it
known: one issue, one fix queued. The reports describe three separate problems, and they need
three fixes. One of them gets worse if only the parser is fixed.

This repo holds the three tickets and one page for the desk's knowledge base.

## The three problems

| Problem | Reports | Why it is its own problem |
|---|---|---|
| The parser reads every date month first | 374 | all 11 tenants write the day first, so day and month trade places |
| `booking-v3` asks for `MM/DD/YYYY` and stores the date day-first | 33 | only through the form, only at the 3 tenants on that form; typed month-first on purpose |
| Invites land one day out | 5 | no second number to trade; one meeting reached two timezones on two different days |

374 + 33 + 5 = 412, so every report is counted once and none is counted twice.

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
- `desk-notes.json` for the third. Those five ids are the desk's own list.

Do not count by `meta.intakeTag`. Intake guesses the cause from the reporter's browser the
moment the report arrives, and its guess disagrees with the problem the report belongs to in
**43 of the 412**.

## How the rendered page was produced

The export ships `src/` only: no `package.json`, no build setup, so mdsvex and Svelte cannot
run against it. The HTML in `rendered/` was produced by a renderer that reads the `.svx` source
itself, so the words have one source and the two files cannot drift apart. The frame, the
component markup and the cause logic are copied from the export verbatim where they are logic.
Build the `.svx` in the real knowledge base and the page is the same page; the HTML here is for
reading it without a build.
