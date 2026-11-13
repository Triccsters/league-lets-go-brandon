<script>
    import { onMount } from 'svelte';
    import data from '$lib/data/dynasty.json';
    import { nameOf } from '$lib/utils/flpHistory';
    import { myTeam } from '$lib/utils/myTeam';
    import YourTeamBadge from '$lib/MyTeam/YourTeamBadge.svelte';

    let ready = false;
    onMount(() => { ready = true; });
    const mine = (h) => ready && h === $myTeam;

    const VIEWS = [['win_now', 'Win now'], ['balanced', 'Balanced'], ['long_term', 'Long term']];
    let view = 'balanced';
    let open = null;
    const max = Math.max(...data.teams.flatMap((t) => [t.win_now, t.balanced, t.long_term]));
    const rankIn = (k) => Object.fromEntries([...data.teams].sort((a, b) => b[k] - a[k]).map((t, i) => [t.h, i + 1]));
    const ranks = { win_now: rankIn('win_now'), balanced: rankIn('balanced'), long_term: rankIn('long_term') };
    $: order = [...data.teams].sort((a, b) => b[view] - a[view]);

    // which way a roster leans: the gap between its win-now and long-term value
    function lean(t) {
        const d = t.win_now - t.long_term;
        if (d >= 5) return ['now', 'Built for now'];
        if (d <= -5) return ['later', 'Built for later'];
        return ['even', 'Even'];
    }
    const when = new Date(data.captured).toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });
    const pickLabel = (p) => `R${p.round}${p.own ? '' : ` (${nameOf(p.from)})`}`;
</script>

<svelte:head><title>Win Now v Long Term</title></svelte:head>

<div class="holder">
    <h1>Win Now v Long Term</h1>
    <p class="sub">Every roster valued three ways: for this season, balanced, and for the years ahead. From {data.source}, read {when}.</p>

    <div class="switch">
        {#each VIEWS as [k, label]}
            <button class:on={view === k} on:click={() => (view = k)}>{label}</button>
        {/each}
    </div>

    <div class="scroll">
        <table>
            <thead><tr><th class="num">#</th><th>Team</th><th class="bar">Win now</th><th class="bar">Balanced</th><th class="bar">Long term</th><th>Lean</th><th class="num">Firsts</th><th class="num">Picks</th><th>Best players</th></tr></thead>
            <tbody>
                {#each order as t, i (t.h)}
                    {@const l = lean(t)}
                    <tr class:mine={mine(t.h)} on:click={() => (open = open === t.h ? null : t.h)}>
                        <td class="num">{i + 1}</td>
                        <td><b>{nameOf(t.h)}</b><YourTeamBadge handle={t.h} size="small" /></td>
                        {#each VIEWS as [k]}
                            <td class="bar"><div class="meter" class:sel={view === k}><span class={k} style="width:{(t[k] / max) * 100}%"></span><em>{t[k].toFixed(1)} <small>#{ranks[k][t.h]}</small></em></div></td>
                        {/each}
                        <td><span class="pill {l[0]}">{l[1]}</span></td>
                        <td class="num">{t.picks.firsts}</td>
                        <td class="num">{t.picks.total}</td>
                        <td class="best">{t.top.slice(0, 3).map((p) => p.n).join(', ')}</td>
                    </tr>
                    {#if open === t.h}
                        <tr class="detail"><td colspan="9">
                            <div class="cols">
                                <div><h3>Top of the roster</h3>{#each t.top as p}<div class="row"><span>{p.n}</span><b class="picks">{p.pos}, {p.age}</b></div>{/each}</div>
                                <div><h3>Worth more right now</h3>{#each t.now_assets as p}<div class="row"><span>{p.n}</span><b class="picks">{p.pos}, {p.age}</b></div>{:else}<p class="none">Nobody stands out.</p>{/each}
                                    <h3 class="gap">Worth more later</h3>{#each t.future_assets as p}<div class="row"><span>{p.n}</span><b class="picks">{p.pos}, {p.age}</b></div>{:else}<p class="none">Nobody stands out.</p>{/each}</div>
                                <div><h3>Value by position</h3>{#each Object.entries(t.pos) as [q, v]}<div class="row"><span>{q}</span><b>{v.toFixed(1)}</b></div>{/each}
                                    <h3 class="gap">Draft picks</h3>{#each data.seasons as s}<div class="row"><span>{s}</span><b class="picks">{(t.picks.by_season[s] || []).map(pickLabel).join(', ') || 'none'}</b></div>{/each}</div>
                            </div>
                        </td></tr>
                    {/if}
                {/each}
            </tbody>
        </table>
    </div>
    <p class="note">Tap a team for its best players, who is worth more now or later, and its draft picks.</p>
    <p class="note">How it works: {data.method} The ranks are the Fantasy Footballers' opinion on the day they were read and are the same for every scoring format, so league scoring is not built in. A pick in brackets came from that manager.</p>
</div>

<style>
    .holder { max-width: 1100px; margin: 0 auto; padding: 1.5em 1em; }
    h1 { text-align: center; margin-bottom: 0.2em; }
    h3 { margin: 0 0 0.4em; font-size: 0.95em; }
    h3.gap { margin-top: 1em; }
    .sub, .note { text-align: center; opacity: 0.75; font-size: 0.92em; }
    .switch { display: flex; justify-content: center; gap: 0.5em; margin: 1em 0 0.4em; }
    .switch button { border: 1px solid rgba(127,127,127,0.5); background: transparent; color: inherit; border-radius: 18px; padding: 0.35em 1.1em; font-weight: 700; cursor: pointer; }
    .switch button.on { background: #3498db; border-color: #3498db; color: #fff; }
    .scroll { overflow-x: auto; margin-top: 0.8em; }
    table { width: 100%; border-collapse: collapse; }
    th, td { padding: 0.45em 0.5em; border-bottom: 1px solid rgba(127,127,127,0.22); text-align: left; font-size: 0.92em; white-space: nowrap; }
    tbody tr { cursor: pointer; }
    tr.mine td { background: rgba(39, 174, 96, 0.15); }
    .num { text-align: right; font-variant-numeric: tabular-nums; }
    .bar { min-width: 130px; }
    .meter { position: relative; height: 1.45em; background: rgba(127,127,127,0.18); border-radius: 4px; overflow: hidden; opacity: 0.6; }
    .meter.sel { opacity: 1; outline: 2px solid rgba(255,255,255,0.55); }
    .meter span { position: absolute; left: 0; top: 0; bottom: 0; }
    .meter span.win_now { background: #c0392b; }
    .meter span.balanced { background: #2980b9; }
    .meter span.long_term { background: #1e8449; }
    .meter em { position: relative; font-style: normal; font-weight: 700; padding-left: 0.4em; line-height: 1.45em; color: #fff; text-shadow: 0 0 3px #000; }
    .meter small { font-weight: 400; opacity: 0.9; }
    .pill { border-radius: 10px; padding: 0.1em 0.6em; font-size: 0.82em; font-weight: 700; color: #fff; }
    .pill.now { background: #c0392b; }
    .pill.later { background: #1e8449; }
    .pill.even { background: #7f8c8d; }
    td.best { white-space: normal; min-width: 260px; font-size: 0.86em; }
    tr.detail td { white-space: normal; background: rgba(127,127,127,0.1); cursor: default; }
    .cols { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 1.2em; padding: 0.5em 0.3em; }
    .row { display: flex; justify-content: space-between; gap: 0.8em; padding: 0.22em 0; border-bottom: 1px solid rgba(127,127,127,0.18); font-size: 0.9em; }
    .row small { opacity: 0.65; }
    .row b { font-variant-numeric: tabular-nums; text-align: right; }
    .row b.picks { font-weight: 400; }
    .none { opacity: 0.6; font-size: 0.88em; margin: 0.2em 0; }
</style>
