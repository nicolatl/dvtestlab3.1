<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "../../lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year)
  let range = Math.max(...years) - Math.min(...years);

  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Bar from '$lib/Bar.svelte';


  let rawData = [];

  let wrangled = [];
  let totalLines = 0;

  onMount(async () => {
      rawData = await d3.json('/lab6_example.json');
      for (let i=0; i< rawData.length; i++) {
        totalLines += rawData[i].lines;
      }
      console.log(totalLines);
  });

  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));
  $: console.log(barData);


</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>
<h1>{projects.length} Projects over {range} Years</h1>
<Bar data={barData}/>


<p>Scroll down to see my a timeline of my projects and how they've contributed to my professional and personal life</p>

<ProjectNarrative />

<p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>

<div class="projects">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<style>
  .outro {
    margin-bottom: 2em;
  }
</style>
