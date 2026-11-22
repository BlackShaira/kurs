<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem that needs its own ticket, counted the way the desk's own view
  // counts, as it was exported. `groupByCause` partitions the ticket into its causes, and
  // `readAs` footnotes the reports the desk read differently without moving them out of
  // the count. So the two cause rows add up to the ticket, and the third problem is a
  // subset of the first: the same reports, read for what their words say.
  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const counted = $derived(groups.reduce((n, g) => n + g.reports.length, 0));

  // The desk's own list, footnoted under the cause its tenant implies.
  const oneDay = $derived(groups.map((g) => readAs(g, notes.oneDayMove)));
  const oneDayTotal = $derived(oneDay.reduce((n, m) => n + m, 0));
  const oneDayUnder = $derived(groups.filter((_, i) => oneDay[i] > 0).map((g) => g.cause).join(', '));

  const WHAT: Record<string, string> = {
    parser: 'the shared parser reads every date month first',
    hint: "booking-v3's date hint says MM/DD/YYYY and the form reads it the tenant's way",
  };
  const channelsIn = (rs: Report[]) =>
    Object.entries(rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.channel]: (n[r.channel] ?? 0) + 1 }), {}))
      .sort((a, b) => b[1] - a[1])
      .map(([c, n]) => `${c} ×${n}`)
      .join(' · ');
  const tags = (g: { taggedAs: Record<string, number> }) =>
    Object.entries(g.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, n]) => `${tag} ×${n}`)
      .join(' · ');
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{groups.length}</strong> {groups.length === 1 ? 'cause' : 'causes'} by the desk's view ·
  <strong>{groups.length + (oneDayTotal ? 1 : 0)}</strong> problems that need a ticket ·
  {counted === merged.length ? 'every report counted once' : `${counted} counted — this does not add up`}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>what it is</th><th>channels</th><th>tagged as</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      <tr>
        <td><code>{g.cause}</code></td>
        <td class="n">{g.reports.length}</td>
        <td>{WHAT[g.cause] ?? 'no cause the rules can name'}</td>
        <td class="muted">{channelsIn(g.reports)}</td>
        <td class="muted">{tags(g)}</td>
        <td class="n">{g.tenants}</td>
        <td class="muted">{g.firstSeen} → {g.lastSeen}</td>
      </tr>
    {/each}
    {#if oneDayTotal}
      <tr class="subset">
        <td><code>one-day</code></td>
        <td class="n">{oneDayTotal}</td>
        <td>the date moves by one day, which is no swap at all</td>
        <td class="muted" colspan="3">counted above under <code>{oneDayUnder}</code>, by tenant, as the desk files them</td>
        <td class="muted">from <code>desk-notes.json</code></td>
      </tr>
    {/if}
  </tbody>
</table>

<p class="note">
  The <code>one-day</code> row is not a fourth share of the {merged.length}. Those
  {oneDayTotal} reports are already inside the <code>{oneDayUnder}</code> count, because the
  desk files a report under the cause its tenant implies and footnotes the ones it read
  differently. They need a ticket all the same: no fix is tracked against them today.
</p>

<p class="note">
  Intake's own tag disagrees with the cause the report belongs to in
  <strong>{groups.reduce((n, g) => n + g.reports.filter((r) => r.meta.intakeTag !== g.cause).length, 0)}</strong>
  of these {merged.length} reports, because intake guesses from the reporter's browser and the
  cause depends on the tenant. Counting by <code>meta.intakeTag</code> gives the wrong number for
  every row above.
</p>

<p class="note">
  The export ships {(reports as Report[]).length} report objects for {merged.length} reports: its
  pages overlap, so some arrive twice. Every count here is after <code>uniqueById</code>.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .n { text-align: right; font-variant-numeric: tabular-nums; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
  .subset td { background: var(--note-bg); }
</style>
