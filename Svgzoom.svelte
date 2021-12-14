<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";
  import { zoom } from "d3-zoom";
  import { select } from "d3-selection";
  import Fa from "svelte-fa/src/fa.svelte";
  import { faUndo } from "@fortawesome/free-solid-svg-icons";

  export let viewBox = "0 0 300 150";
  export let translateVar = [-30, -5, 1];

  let svg, g;
  onMount(() => {
    if (svg && g) {
      select(svg).call(zoomObj);
    }
  });

  function resetZoom() {
    select(svg)
      .transition()
      .duration(0)
      .call(zoomObj.transform, d3.zoomIdentity);
  }

  let zoomObj = d3
    .zoom()
    .scaleExtent([1, 10])
    .on("zoom", ({ transform }) => {
      translateVar[0] = transform.x;
      translateVar[1] = transform.y;
      translateVar[2] = transform.k;
      const { k, x, y } = transform;
      select(g).attr("transform", `translate(${x}, ${y}) scale(${k})`);
    });

  $: buttonZoom = [translateVar[0], translateVar[1]];
</script>

<svg viewBox={viewBox} bind:this={svg}>
  <g bind:this={g}>
    <slot></slot>
  </g>
</svg>
<button id="reset" on:click={resetZoom}><Fa icon={faUndo} /></button>

<style>
  button {
    position: absolute;
    right: 10px;
    top: 10px;
    height: max-content;
    width: max-content;
    padding: 5px 6px;
    background-color: hsla(0, 50%, 50%, 0.2);
    border: none;
    border-radius: 5px;
    color: #c9082a;
    box-shadow: rgba(9, 30, 66, 0.25) 0px 4px 8px -2px,
      rgba(9, 30, 66, 0.08) 0px 0px 0px 1px;
  }

  button:hover {
    border: 1px solid black;
    cursor: pointer;
  }

  svg {
    background-color: lightcyan;
    box-shadow: rgba(0, 0, 0, 0.05) 0px 0px 0px 1px,
      rgb(209, 213, 219) 0px 0px 0px 1px inset;
  }
</style>
