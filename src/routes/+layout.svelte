<script>
    import { base } from "$app/paths";
    import { page } from "$app/stores";
    import "../style.css";
    


    let pages = [
        {url: "/", title: "About"},
        {url: "/projects", title: "Projects"},
        {url: "/resume", title: "Resume"},
        {url: "/contact", title: "Contact"},
        {url: "https://github.com/nicolatl", title: "Github"}
    ]
    let colorScheme = "light dark"; // default colorScheme value

    let localStorage = globalThis.localStorage ?? {};


    if (localStorage.colorScheme) { // if localStorage has a colorScheme property
        colorScheme = localStorage.colorScheme; // override the default colorScheme
    }

    let root = globalThis?.document?.documentElement;
    $: root?.style.setProperty("color-scheme", colorScheme);

    $: localStorage.colorScheme = colorScheme;

</script>

<label class="color-scheme-switch">
    Theme:
    <select bind:value={ colorScheme }>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<nav>
    {#each pages as p}
        <a href={base + p.url}
            class:current={p.url === "/" // is this link the home page?
            ? $page.url.pathname === (base + "/") // if yes - set current = true if the path name matches. Else, set current = true if the path name starts correctly
            : $page.url.pathname.startsWith(base + p.url)}
            target={p.url.startsWith("http") ? "_blank": null}
        >
            {p.title}
        </a>


    {/each}
</nav>
<slot />
<style>
    .color-scheme-switch {
        position: absolute;
        top: 1rem;
        right: 1rem;
        display: inline-flex;
        gap: 4px;
    }
    :root {
        color-scheme: dark;
    }
    nav {
        --border-color: oklch(50% 10% 200 / 40%);
        display: flex;
        margin-bottom: 1rem;
        border-bottom: 1px solid var(--border-color);
    }

    nav a {
        flex: 1;
        text-decoration: none;
        color: inherit;
        text-align: center;
        padding: 0.5em;
    }

    nav a.current {
    border-bottom: 0.4em solid var(--border-color);
    padding-bottom: 0.1em;
    }

    nav a:hover {
    border-bottom: 0.4em solid var(--color-accent);
    padding-bottom: 0.1em;
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);

    }
</style>