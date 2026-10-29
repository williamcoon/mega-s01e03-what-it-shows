<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { causeOf, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem in the ticket. A report the desk read as a one-day move is its own problem,
  // not a footnote under its tenant's cause.
  let { ticket, issues }: { ticket: number; issues: Record<string, string> } = $props();

  const PROBLEMS = [
    { key: 'parser', what: 'parser reads day-first dates month first' },
    { key: 'hint', what: 'booking-v3 hint says month first, form reads day first' },
    { key: 'one-day', what: 'date moves one day, not swapped' },
  ] as const;

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const problemOf = (r: Report) =>
    notes.oneDayMove.includes(r.id) ? 'one-day' : (causeOf(r, tenantOf(r)) ?? 'unplaced');
  const rows = $derived(
    PROBLEMS.map((p) => {
      const rs = merged.filter((r) => problemOf(r) === p.key);
      const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
      return { ...p, reports: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, first: days[0], last: days[days.length - 1] };
    }),
  );
  const unplaced = $derived(merged.filter((r) => problemOf(r) === 'unplaced').length);
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.filter((r) => r.reports).length}</strong> problems
  {#if unplaced}· <strong>{unplaced}</strong> unplaced{/if}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.key)}
      <tr>
        <td><code>{r.key}</code> {r.what}</td>
        <td>{r.reports}</td>
        <td>{r.tenants}</td>
        <td>{r.first} → {r.last}</td>
        <td>{#if issues[r.key]}<a href={issues[r.key]}>{issues[r.key].replace(/^.*\/issues\//, '#')}</a>{:else}none{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
