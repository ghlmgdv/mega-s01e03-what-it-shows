<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type CauseGroup, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  const uniqueReports = uniqueById(reports as Report[]).filter((r) => r.mergedInto === 4471);
  const groups = groupByCause(uniqueReports, tenantOf);
  const channelCounts = (g: CauseGroup) => {
    const counts = g.reports.reduce<Record<string, number>>((n, r) => {
      n[r.channel] = (n[r.channel] ?? 0) + 1;
      return n;
    }, {});
    return Object.entries(counts).map(([channel, count]) => `${channel} ×${count}`).join(' · ');
  };
  const tags = (g: CauseGroup) =>
    Object.entries(g.taggedAs)
      .sort((a, b) => b[1] - a[1])
      .map(([tag, count]) => `${tag} ×${count}`)
      .join(' · ');
</script>

<p class="summary">
  <strong>{uniqueReports.length}</strong> unique reports merged into #4471 from
  <strong>{reports.length}</strong> export rows · <strong>{groups.length}</strong> distinct causes.
</p>

<table>
  <thead>
    <tr>
      <th>cause</th>
      <th>reports</th>
      <th>tenants</th>
      <th>channels</th>
      <th>intake tag (not cause)</th>
      <th>received</th>
    </tr>
  </thead>
  <tbody>
    {#each groups as group (group.cause)}
      <tr>
        <td><code>{group.cause}</code></td>
        <td>{group.reports.length}</td>
        <td>{group.tenants}</td>
        <td>{channelCounts(group)}</td>
        <td>{tags(group)}</td>
        <td>{group.firstSeen} → {group.lastSeen}</td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="note">
  The desk noted {readAs(groups.find((group) => group.cause === 'parser')!, notes.oneDayMove)}
  one-day-move reports; tenant and form data still place them under <code>parser</code>.
  Intake tags are guesses and can disagree with the cause.
</p>

<style>
  .summary { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.9rem; }
</style>
