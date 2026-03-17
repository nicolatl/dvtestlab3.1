<script>
    import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';
    import BarHorizontal from '../../lib/BarHorizontal.svelte';
import {
	computePosition,
	autoPlacement,
	offset,
} from '@floating-ui/dom';


let locData = [];
let barData = [];
let commits = [];

let width = 1000, height = 600;
let margin = {top: 10, right: 10, bottom: 30, left: 20};
let usableArea = {
	top: margin.top,
	right: width - margin.right,
	bottom: height - margin.bottom,
	left: margin.left
};
usableArea.width = usableArea.right - usableArea.left;
usableArea.height = usableArea.bottom - usableArea.top;


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
            totalLines: lines.length,
            lines: lines
        };


        return ret;
    });
    commits = d3.sort(commits, d => -d.totalLines);


    barData = d3.rollups(locData, v => v.length, d => d.type)
    .map(([type, count]) => ({ label: String(type), value: count }));
});

$: selectedLines = (clickedCommits.length > 0 ? clickedCommits.flatMap(d => d.lines) : locData);
$: selectedCounts = d3.rollup(
    selectedLines,
    v => v.length,
    d => d.type
);
$: allTypes = Array.from(new Set(locData.map(d => d.type)));
$: barData = allTypes.map(type =>  ({ label: type, value: selectedCounts.get(type) || 0 }));


// $: barData = d3.rollups(locData, v => v.length, d => d.type)
//     .map(([type, count]) => ({ label: String(type), value: count }));

// $: console.log(languageBreakdown);
$: console.log(barData);

// Thanks to Nathanael Jenkins for flagging this to us!
// $: minDate = d3.min(commits.map(d => d.date));
// $: maxDate = d3.max(commits.map(d => d.date));
$: [minDate, maxDate] = d3.extent(commits.map(d => d.date));
$: maxDatePlusOne = new Date(maxDate);
$: maxDatePlusOne.setDate(maxDatePlusOne.getDate() + 1);

$: xScale = d3.scaleTime()
              .domain([minDate, maxDatePlusOne])
              .range([usableArea.left, usableArea.right])
              .nice();

$: yScale = d3.scaleLinear()
              .domain([24, 0])
              .range([usableArea.bottom, usableArea.top]);
$: rScale = d3.scaleSqrt()
              .domain(d3.extent(commits.map(d => d.totalLines)))
              .range([5,30]);

let xAxis, yAxis;
$: {
	d3.select(xAxis).call(d3.axisBottom(xScale));
	d3.select(yAxis).call(d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, "0") + ":00"));
}

let yAxisGridlines;

$: {
	d3.select(yAxisGridlines).call(
		d3.axisLeft(yScale)
		  .tickFormat("")
		  .tickSize(-usableArea.width)
	);
}

let hoveredIndex = -1;
$: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

let cursor = {x: 0, y: 0};

let commitTooltip;

let tooltipPosition = {x: 0, y: 0};
async function dotInteraction (index, evt) {
	let hoveredDot = evt.target;
	if (evt.type === "mouseenter") {
		hoveredIndex = index;
		cursor = {x: evt.x, y: evt.y};
		tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
			strategy: "fixed", // because we use position: fixed
			middleware: [
				offset(5), // spacing from tooltip to dot
				autoPlacement() // see https://floating-ui.com/docs/autoplacement
			],
		});        }
	else if (evt.type === "mouseleave") {
		hoveredIndex = -1
	}
    else if (evt.type === "click") {
        let commit = commits[index]
        if (!clickedCommits.includes(commit)) {
            // Add the commit to the clickedCommits array
            clickedCommits = [...clickedCommits, commit];
        }
        else {
                // Remove the commit from the array
                clickedCommits = clickedCommits.filter(c => c !== commit);
        }
        console.log(clickedCommits);
    }
    
}

let clickedCommits = [];




</script>

<svelte:head>
  <title>Meta</title>
</svelte:head>
<h1>Meta</h1>
<p>Meta page to visualize project data</p>
<h3>Commits by time of day</h3>
<svg viewBox="0 0 {width} {height}">
    <g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
    <g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
    <g class="gridlines" transform="translate({usableArea.left}, 0)" bind:this={yAxisGridlines} />
	<g class="dots">
    {#each commits as commit, index }
        <circle      
            class:selected={ clickedCommits.includes(commit) }     
            on:mouseenter={evt => dotInteraction(index, evt)}
            on:mouseleave={evt => dotInteraction(index, evt)}
            on:click={ evt => dotInteraction(index, evt) }
            cx={ xScale(commit.datetime) }
            cy={ yScale(commit.hourFrac) }
            r={ rScale(commit.totalLines) }
            fill="steelblue"
        />
    {/each}
    </g>

</svg>

<dl class="info tooltip" hidden={hoveredIndex === -1} style="top: {tooltipPosition.y}px; left: {tooltipPosition.x}px" bind:this={commitTooltip}>
	<dt>Commit</dt>
	<dd><a href="{ hoveredCommit.url }" target="_blank">{ hoveredCommit.id }</a></dd>
    
	<dt>Author</dt>
	<dd>{ hoveredCommit.author }</dd>

	<dt>Date</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleString("en", {dateStyle: "full"}) }</dd>

    <dt>Time</dt>
	<dd>{ hoveredCommit.datetime?.toLocaleString("en", {timeStyle: "short"}) }</dd>

	<dt>Lines</dt>
	<dd>{ hoveredCommit.totalLines }</dd>
</dl>


<BarHorizontal data={barData} title={clickedCommits.length > 0 ? "Lines of Code: Selected commits" : "Lines of Code: Website Breakdown"}/>

<style>
	svg {
		overflow: visible;
	}
    .gridlines {
        stroke-opacity: .2;
    }
    circle {
        fill-opacity: .7;
        transition: 200ms;
        &:hover {
		    fill:darkgreen;
	    }
    }
    dl.info {
        display: grid;
        font-size:smaller;
        transition-duration: 500ms;
        transition-property: opacity, visibility;

        &[hidden]:not(:hover, :focus-within) {
            opacity: 0;
            visibility: hidden;
        }
    }
    
    dt {
        grid-column: 1;
        color:grey;
    }
    dd {
        grid-column: 2;
        margin-left: 12px;
    }
    .tooltip {
        position: fixed;
        top:1em;
        left:1em;
        background-color: oklch(100% 0% 0 / 80%);
        box-shadow: 1px 1px 6px 1px lightgrey;
        border-radius: 4px;
        backdrop-filter: blur(10px);
        padding: 4px;
    }
    .selected {
        fill: var(--color-accent);
    }

</style>
