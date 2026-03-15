<script>
    import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';
    import BarHorizontal from '../../lib/BarHorizontal.svelte';

let locData = [];
let barData = [];
let commits = [];


onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth),
        date: new Date(row.date + "T00:00" + row.timezone),
        datetime: new Date(row.datetime)
    }));
    
    //lab 7 insertions are here
    commits = d3.groups(locData, d => d.commit);
    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
        let first = lines[0];
        let {author, date, time, timezone, datetime} = first;
        let ret = {
            id: commit,
            url: "https://github.com/vis-society/lab-7/commit/" + commit,
            author, date, time, timezone, datetime,
            hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
            totalLines: lines.length
        };

        // Like ret.lines = lines
        // but non-enumerable so it doesn’t show up in JSON.stringify
        Object.defineProperty(ret, "lines", {
            value: lines,
            configurable: true,
            writable: true,
            enumerable: false,
        });

        return ret;
    });
    console.log(commits);


    barData = d3.rollups(locData, v => v.length, d => d.type)
    .map(([type, count]) => ({ label: String(type), value: count }));
    console.log(barData);
});

$: barData = d3.rollups(locData, v => v.length, d => d.type)
    .map(([type, count]) => ({ label: String(type), value: count }));


</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>Meta page to visualize project data</p>
{JSON.stringify(commits, null, "\t")}
<BarHorizontal data={barData}/>