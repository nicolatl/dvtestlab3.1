<script>
    export let data = {};
</script>

<article>
  <img src={data.image} alt={data.title} />

  <div>
    <div class="title">{data.title}</div>
    <div class="author">{data.author}</div>
  </div>
</article>

<style>
    img {
        height: 6em;
    }
    .title {
        font-weight: bold;
    }
    article {
        display: flex;
        align-items: center;
        gap: 1em;
        margin-bottom: 1em;
    }
</style>