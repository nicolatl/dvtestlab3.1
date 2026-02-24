<script>
    function handleSubmit(event) {
        event.preventDefault();
        let form = event.target;
        let data = new FormData(form);
        let url = form.action + "?";
        for (let [name, value] of data) {
	        url += (name + "=" + value + "&");
        }
        location.href = url;

    }

</script>

<svelte:head>
  <title>Contact me</title>
</svelte:head>

<h1>Contact</h1>
Please do not contact me in this way.
<form action="mailto:77587217+nicolatl@users.noreply.github.com" method="GET" on:submit={handleSubmit}>
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
