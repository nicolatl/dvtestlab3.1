<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>
<nav>
    <a href=".">Home</a>
    <a href="projects">Projects</a>
    <a class="current" href="contact">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/nicolatl" target="_blank">GitHub</a>
</nav>
Please do not contact me in this way.
<form action="mailto:77587217+nicolatl@users.noreply.github.com" method="GET">
    <label>
        Email:
        <input type="email" name="email" required>
    </label>
    <br>
    <label>
        Subject:
        <input name="subject" required>
    </label>
    <br>
    <label>
        Message:
        <textarea name="body" required></textarea>
    </label>
    <br>
    <button>Send</button>
</form>
