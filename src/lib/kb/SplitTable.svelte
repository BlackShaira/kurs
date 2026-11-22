<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem the reports actually describe, not per cause the rules can name.
  // `causes.ts` knows two causes, parser and hint, and it reads them off the tenant and the
  // channel. It cannot see the third problem, because that one is only in the words: the
  // desk listed those reports in desk-notes.json and filed them under the cause their
  // tenant implies. Here they are counted on their own, so the three counts add up to the
  // reports merged into the ticket and nothing is counted twice.
  let { ticket }: { ticket: number } = $props();

  type Row = {
    key: 'parser' | 'hint' | 'one-day';
    what: string;
    reports: Report[];
  };

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));

  const rows = $derived.by<Row[]>(() => {
    const oneDay = merged.filter((r) => notes.oneDayMove.includes(r.id));
    const rest = merged.filter((r) => !notes.oneDayMove.includes(r.id));
    return [
      {
        key: 'parser',
        what: 'the shared parser reads every date month first',
        reports: rest.filter((r) => causeOf(r, tenantOf(r)) === 'parser'),
      },
      {
        key: 'hint',
        what: "booking-v3's date hint says MM/DD/YYYY and the form reads it the tenant's way",
        reports: rest.filter((r) => causeOf(r, tenantOf(r)) === 'hint'),
      },
      {
        key: 'one-day',
        what: 'the date moves by one day, which is no swap at all',
        reports: oneDay,
      },
    ];
  });

  const counted = $derived(rows.reduce((n, r) => n + r.reports.length, 0));
  const tenantsIn = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;
  const channelsIn = (rs: Report[]) =>
    Object.entries(rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.channel]: (n[r.channel] ?? 0) + 1 }), {}))
      .sort((a, b) => b[1] - a[1])
      .map(([c, n]) => `${c} ×${n}`)
      .join(' · ');
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
  // What intake guessed for these reports, which is not where they belong.
  const taggedWrong = (rs: Report[], key: Row['key']) =>
    rs.filter((r) => r.meta.intakeTag !== (key === 'one-day' ? null : key)).length;
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ·
  <strong>{rows.length}</strong> separate problems ·
  {counted === merged.length ? 'every report counted once' : `${counted} counted — this does not add up`}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>what it is</th><th>channels</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.key)}
      <tr>
        <td><code>{row.key}</code></td>
        <td class="n">{row.reports.length}</td>
        <td>{row.what}</td>
        <td class="muted">{channelsIn(row.reports)}</td>
        <td class="n">{tenantsIn(row.reports)}</td>
        <td class="muted">{seen(row.reports)}</td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  Intake's own tag disagrees with the problem the report belongs to in
  <strong>{rows.reduce((n, r) => n + taggedWrong(r.reports, r.key), 0)}</strong>
  of these {merged.length} reports, because intake guesses from the reporter's browser and the
  problem depends on the tenant. Counting by <code>meta.intakeTag</code> gives the wrong number
  for every row above.
</p>

<p class="note">
  The export ships {(reports as Report[]).length} report objects for {merged.length} reports: its
  pages overlap, so some arrive twice. Every count here is after
  <code>uniqueById</code>.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .n { text-align: right; font-variant-numeric: tabular-nums; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
