<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import books from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<h1>Nicola</h1>
<nav>
    <a class="current" href=".">Home</a>
    <a href="projects">Projects</a>
    <a href="contact">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/nicolatl" target="_blank">GitHub</a>
</nav>

<div class="top-matter">
    <div class="about">
        <p>I am a masters student in Technology and Policy and Computer Science. I currently TA Interactive Data Visualization & Society.</p>
        <img src="images/nicola.jpeg" width="500" alt="Nicola smiling and holding an umbrella in front of a cat">
    </div>

    <div class="reading">
        <h2>What I'm Reading</h2>
        {#each books as b}
        <ReadingItem data={b} />
        {/each}
    </div>
</div>

<h2>Latest projects</h2>
<div class="projects">
    {#each projects.slice(0, 3) as p}
    <Project data={p} />
    {/each}
</div>

<style>
    .reading {
        background-color: oklch(80% 3% 200);
        padding-left: 1em;
        padding-right: 1em;
        padding-bottom: -1em;
        gap: 1em;
        align-items: center;
    }
    .top-matter {
        display: flex;
        gap: 1em;
    }
    .about {
        max-width: 500px;
    }
</style>
