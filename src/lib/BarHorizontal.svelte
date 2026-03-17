<script>
    import * as d3 from 'd3';

    let width = 400;
    let height = 100;

    export let data = [];
    export let title = "";



    let margin = { top: 20, right: 60, bottom: 30, left: 60 };
    let innerWidth  = width  - margin.left - margin.right;
    let innerHeight = height - margin.top  - margin.bottom;

    $: yScale = d3.scaleBand()
        .domain(data.map(d => d.label))
        .range([0, innerHeight])
        .padding(0.2);

    $: xScale = d3.scaleLinear()
        .domain([0, d3.max(data, d => d.value) || 1])
        .range([0, innerWidth]);

    $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
        .domain(data.map(d => d.label));

    let xAxis, yAxis;

    $: if (xAxis && yAxis) {
        d3.select(xAxis).call(
            d3.axisBottom(xScale)
            .ticks(Math.min(d3.max(data, d => d.value), 10))
        );
        d3.select(yAxis).call(d3.axisLeft(yScale));
    }

    $: maxBar = d3.greatest(data, d => d.value);


</script>
<div class="container">
    <svg viewBox="0 0 {width} {height}">
    <text
        x={margin.left + innerWidth / 2}
        y={margin.top / 2}
        text-anchor="middle"
        class="chart-title">
        {title}
    </text>
    <!-- x-axis label -->
    <text
        x={margin.left + innerWidth / 2}
        y={innerHeight + margin.bottom + 20}
        text-anchor="middle"
        class="axis-label">
        Lines of Code
    </text>

    <!-- y-axis label -->
    <text
        x={-margin.top - (innerHeight / 2)}
        y={0}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        <tspan>Programming</tspan>
        <tspan x={-margin.top - (innerHeight / 2)} dy=1em>Language</tspan>
    </text>

    <g transform="translate({margin.left}, {margin.top + innerHeight})"
       bind:this={xAxis} />
    <g transform="translate({margin.left}, {margin.top})"
       bind:this={yAxis} />
    <g transform="translate({margin.left}, {margin.top})">
        {#each data as d}
            <rect
                y={yScale(d.label)}
                x={0}
                height={yScale.bandwidth()}
                width={xScale(d.value)}
                fill={colorScale(d.label)}
            />
        {/each}
        {#if maxBar}
            <!-- highlight outline around the tallest bar -->
            <rect
                x={0}
                y={yScale(maxBar.label)}
                width={xScale(maxBar.value)}
                height={yScale.bandwidth()}
                fill="none"
                stroke="currentColor"
                stroke-width="2"
            />
            <!-- leader line -->
            <!-- <line
                x1={xScale(maxBar.value) / 2}
                y1={yScale(maxBar.label)}
                x2={xScale(maxBar.value) / 2}
                y2={yScale(maxBar.label)-30}
                stroke="currentColor"
                stroke-width="1"
            /> -->
            <!-- annotation text at end of leader line -->
            <text
                x={xScale(maxBar.value) + 6}
                y={yScale(maxBar.label) + yScale.bandwidth() / 2}
                dominant-baseline="middle"
                text-anchor="start"
                class="annotation">
                Most lines of code
            </text>
        {/if}

    </g>
    </svg>
    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>

<style>
    svg {
        max-width: 100%;
        height: auto;
        overflow: visible;
    }
    .container {
        display:flex;
    }
    .legend {
        flex: 1;
    }
    .swatch {
        background-color: var(--color);
        width: 12px;
        height: 12px;
    }
    li {
        display: flex;
        align-items: center;
        gap:4px;
    }
    .chart-title {
        font-size: 0.6em;
        font-weight: bold;
        fill: currentColor;
    }
    .axis-label {
        font-size: 0.5em;
    }
    .annotation {
        font-size: 0.5em;
        fill: black;
        font-style: italic;
    }

</style>