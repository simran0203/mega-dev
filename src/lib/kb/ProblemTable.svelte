<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { PROBLEM_LABEL, problemRows, type Problem } from '#lib/kb/problems';

  // `issues`: problem → URL of the ticket filed for it.
  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<Problem, string>> } = $props();

  const result = $derived(problemRows(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const channels = (c: Record<string, number>) =>
    Object.entries(c)
      .sort((a, b) => b[1] - a[1])
      .map(([k, n]) => `${k} ×${n}`)
      .join(' · ');
  const num = (url: string) => '#' + url.split('/').pop();
</script>

<p class="sum">
  <strong>{result.total}</strong> reports merged into #{ticket} ·
  <strong>{result.rows.length}</strong>
  {result.rows.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>came in by</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each result.rows as r (r.problem)}
      <tr>
        <td><code>{r.problem}</code> {PROBLEM_LABEL[r.problem]}</td>
        <td>{r.reports}</td>
        <td>{r.tenants}</td>
        <td class="muted">{channels(r.channels)}</td>
        <td>{r.firstSeen} → {r.lastSeen}</td>
        <td>
          {#if issues[r.problem]}<a href={issues[r.problem]}>{num(issues[r.problem]!)}</a>{:else}<span class="none">none</span>{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .none { color: var(--warn); font-weight: 600; }
</style>
