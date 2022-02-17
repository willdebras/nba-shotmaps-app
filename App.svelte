<script>
  import Charts from "./Charts.svelte";
  import Shotmap from "./Shotmap.svelte";
  import Svgzoom from "./Svgzoom.svelte";
</script>

<svelte:head>
<style>
  @import url("https://fonts.googleapis.com/css?family=Graduate&display=swap");
  @import url("https://fonts.googleapis.com/css?family=Roboto&display=swap");
</style>
</svelte:head>

<style>
  main {
    font-family: sans-serif;
    text-align: center;
    padding: 20px;
    font-family: "Roboto";
  }

  :root {
    background-color: #f4f5f2;
    font-family: "Roboto";
  }
</style>

<main>
	<Charts />
</main>