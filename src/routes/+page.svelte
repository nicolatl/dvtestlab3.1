<script>
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";
    import books from "$lib/reading.json";
    import ReadingItem from "$lib/ReadingItem.svelte";

    import { onMount } from "svelte";

    let githubData = null; // This will eventually hold our Github stats
    let loading = true; // This will be true *until* the fetch's promise resolves to a value
    let error = null; // If the API call resulted in an error, it will go into this variable

    onMount(async () => {
        try { // First, try running this block of code
            console.log("Page has been mounted!")
            let response = await fetch("https://api.github.com/users/nicolatl");
            console.log(response);
            githubData = await response.json();
            console.log(githubData);
        } catch (err) { // if the "try" block runs into an error, cancel excecution and run this code instead
            error = err;
        }
        loading = false; // don't forget to add this line!
    })


</script>

<h1>Nicola</h1>

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
<section>
    <h2>GitHub Stats</h2>
    {#if loading}
        <p>Loading...</p>
    {:else if error}
        <p>Something went wrong: {error.message}</p>
    {:else}
        <dl>
            <dt>Followers</dt>
            <dd>{githubData.followers}</dd>
            <dt>Following</dt>
            <dd>{githubData.following}</dd>
            <dt>Public Repositories</dt>
            <dd>{githubData.public_repos}</dd>
        </dl>
    {/if}

</section>

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
    dl {
        display: grid;
    }
    dt {
        grid-row: 1;
    }
    dd {
        margin:0;
    }
</style>
