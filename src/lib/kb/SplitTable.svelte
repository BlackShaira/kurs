<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // What the ticket held, and where those reports are now.
  //
  // This reads every report in the export, not only the ones that still point at this
  // ticket, because that is the point of the page: all of them arrived here, and one
  // cause has since been moved to the ticket it should have had. `CauseTable` shows the
  // ticket as it stands; this shows the split that produced it.
  //
  // The third row is not a cause. `causes.ts` separates two cases: reports whose cause is
  // not the ticket's cause were "merged into the wrong place", and reports the desk read
  // differently are "counted where they're filed, footnoted under it". The first case got
  // a ticket. The second stays where it is, and is a subset of the row above it.
  let { ticket }: { ticket: number } = $props();

  const all = $derived(uniqueById(reports as Report[]));

  type Row = { key: string; what: string; reports: Report[]; subsetOf?: string };

  const rows = $derived.by<Row[]>(() => {
    const byCause = new Map<string, Report[]>();
    for (const r of all) {
      const c = causeOf(r, tenantOf(r)) ?? 'unplaced';
      byCause.set(c, [...(byCause.get(c) ?? []), r]);
    }
    const what: Record<string, string> = {
      parser: 'the shared parser reads every date month first',
      hint: "booking-v3's date hint says MM/DD/YYYY and the form reads it the tenant's way",
    };
    const out: Row[] = [...byCause.entries()]
      .sort((a, b) => b[1].length - a[1].length)
      .map(([key, rs]) => ({ key, what: what[key] ?? 'no cause the rules can name', reports: rs }));
    const oneDay = all.filter((r) => notes.oneDayMove.includes(r.id));
    if (oneDay.length) {
      const host = out.find((row) => row.reports.some((r) => oneDay.includes(r)))?.key;
      out.push({
        key: 'one-day',
        what: 'the date moves by one day, which is no swap at all',
        reports: oneDay,
        subsetOf: host,
      });
    }
    return out;
  });

  const counted = $derived(rows.filter((r) => !r.subsetOf).reduce((n, r) => n + r.reports.length, 0));
  const ticketsOf = (rs: Report[]) =>
    [...new Set(rs.map((r) => r.mergedInto))].sort((a, b) => Number(a) - Number(b));
  const channelsIn = (rs: Report[]) =>
    Object.entries(rs.reduce<Record<string, number>>((n, r) => ({ ...n, [r.channel]: (n[r.channel] ?? 0) + 1 }), {}))
      .sort((a, b) => b[1] - a[1])
      .map(([c, n]) => `${c} ×${n}`)
      .join(' · ');
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
  const taggedWrong = $derived(
    rows
      .filter((r) => !r.subsetOf)
      .reduce((n, row) => n + row.reports.filter((r) => r.meta.intakeTag !== row.key).length, 0),
  );
</script>

<p class="sum">
  <strong>{counted}</strong> reports arrived in #{ticket} ·
  <strong>{rows.filter((r) => !r.subsetOf).length}</strong> causes ·
  <strong>{rows.length}</strong> problems ·
  {counted === all.length ? 'every report counted once' : `${all.length} in the export — this does not add up`}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>ticket</th><th>what it is</th><th>channels</th><th>tenants</th><th>seen</th></tr>
  </thead>
  <tbody>
    {#each rows as row (row.key)}
      <tr class:subset={row.subsetOf}>
        <td><code>{row.key}</code></td>
        <td class="n">{row.reports.length}</td>
        <td>
          {#each ticketsOf(row.reports) as t, i (t)}{i ? ', ' : ''}#{t}{/each}
          {#if row.subsetOf}<span class="muted"> — inside <code>{row.subsetOf}</code></span>{/if}
        </td>
        <td>{row.what}</td>
        <td class="muted">{channelsIn(row.reports)}</td>
        <td class="n">{new Set(row.reports.map((r) => r.tenant)).size}</td>
        <td class="muted">{seen(row.reports)}</td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  The <code>one-day</code> row is a subset of the row it sits under, not a share of its own, so the
  causes above it are what add up. Those reports are in the right ticket and read the wrong way:
  <code>readAs</code> counts them where they are filed and footnotes them there, and the desk keeps
  the list in <code>desk-notes.json</code>.
</p>

<p class="note">
  Intake's own tag disagrees with the cause the report belongs to in <strong>{taggedWrong}</strong>
  of these {counted} reports, because intake guesses from the reporter's browser and the cause
  depends on the tenant. Counting by <code>meta.intakeTag</code> gives the wrong number for every
  row above.
</p>

<p class="note">
  The export ships {(reports as Report[]).length} report objects for {all.length} reports: its
  pages overlap, so some arrive twice. Every count here is after <code>uniqueById</code>.
</p>

<style>
  .sum { font-size: 1.1rem; }
  .n { text-align: right; font-variant-numeric: tabular-nums; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .note { color: var(--muted); font-size: 0.85em; }
  .subset td { background: var(--note-bg); }
</style>
