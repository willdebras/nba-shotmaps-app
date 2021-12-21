<script>
  import * as d3 from "d3";
  import courtpoints from "./public/courtpoints.json";
  import courtnested from "./public/courtnested.json";
  import allshots from "./public/test.json";
  import { fade, fly } from "svelte/transition";
  import { elasticOut } from "svelte/easing";
  import ZoomSvg from "./Svgzoom.svelte";

  let wrapper;
  let height = 800;
  $: width = height;

  $: yAccessor = d => d.y;
  $: xAccessor = d => d.x;

  let yScale = d3
    .scaleLinear()
    .domain(d3.extent(courtpoints, yAccessor))
    .range([height, 0]);

  //const xScale = d3.scaleTime()
  let xScale = d3
    .scaleLinear()
    .domain(d3.extent(courtpoints, xAccessor))
    .range([0, width]);

  let lineGenerator = d3
    .line()
    .x(d => xAccessor(d))
    .y(d => yAccessor(d));

  const filterData = function(value) {
    if (value.length > 0) {
      const id = value[0];
      const obj_shots = allshots.filter(d => d.game_id === id);
      const idd_shots = obj_shots.map((x, i) => {
        x.id = i + 1;
        return x;
      });
      return idd_shots;
    } else {
      return [];
    }
  };

  $: shotsquery = filterData(gameinfo);
  $: shotnumbers = Array(shotsquery.length)
    .fill()
    .map((element, index) => index);

  const debug = function(value) {
    if (shotsquery.length > 0) {
      console.log(shotsquery);
    }
  };

  let colorScale = d3
    .scaleOrdinal()
    .domain(["Made Shot", "Missed Shot"])
    .range(["#ff8c00", "#d3d3d3"]);

  let widthScale = d3
    .scaleOrdinal()
    .domain(["Made Shot", "Missed Shot"])
    .range([0.2, 0.1]);

  let tooltipBody = "";
  let xPos = 0;
  let yPos = 0;
  let opac = 0;
  let league_av = "";

  $: yScaleTooltip = d3
    .scaleLinear()
    .domain([-5, 55])
    .range([0, height]);

  $: xScaleTooltip = d3
    .scaleLinear()
    .domain([-30, 30])
    .range([0, width]);

  let calculateTooltip = function(d) {
    if (shotsquery.length > 0) {
      tooltipBody =
        "<span style = 'color:#17408b; font-weight:700;'>" +
        d.player_name +
        "</span> " +
        "<i>" +
        (d.event_type === "Made Shot" ? "made" : "missed") +
        "</i> this " +
        d.shot_distance +
        "ft. " +
        d.action_type.toLowerCase() +
        " in period " +
        d.period +
        " with " +
        d.minutes_remaining +
        " minutes, " +
        d.seconds_remaining +
        " seconds remaining.";

      xPos = xScaleTooltip(d.loc_x);
      yPos = yScaleTooltip(d.loc_y);
      opac = 1;
      league_av = d.fg_pct;
    }
  };

  let translateVar = [0, 0, 1];

  $: scaleZoom = translateVar[2];
  $: xPosZoom =
    scaleZoom == 1 ? xPos + xScaleTooltip(translateVar[0] / scaleZoom - 30) : 60;
  $: yPosZoom = scaleZoom == 1 ? yPos + yScaleTooltip(translateVar[1] - 5) : 100;

  var emptyTooltip = function() {
    tooltipBody = "";
    opac = 0;
    league_av = "";
  };

  function transitionCircle(node, { delay = 0, duration = 400 }) {
    const o = +getComputedStyle(node).r;

    return {
      delay,
      duration,
      css: t => `stroke-width: ${t * o}`
    };
  }

  // $: points = shotsquery.map(shot => {
  //   return [shot.loc_x, shot.loc_y];
  // });

  // $: delaunay = d3.Delaunay.from(points);
  // $: voronoi = delaunay.voronoi([0, 0, width, height]);

  // let hoveredShot;

  //$: tooltipBody = calculateTooltip(shotsquery[0]);

  // hot metrics

  export let paints = Object.keys(courtnested);
  export let gameinfo;
  export let team1;
  export let team2;
</script>

{#if team2}
<h2>
{team1} v. {team2}
</h2>
{/if}

<div class="wrapper" on:mousemove={(e)=>{
    const bounds = e.target.getBoundingClientRect()
    const x = e.clientX - bounds.left
    const y = e.clientY - bounds.top
    //const pointIndex = delaunay.find(x,y)
    //console.log(pointIndex)
    //hoveredShot = points[pointIndex]
    }} bind:clientHeight={height} style="width: {width}px;">


  <div class = "bleachers" id="bleachers-left"></div>

  <ZoomSvg bind:translateVar={translateVar} viewBox = "-30 -5 60 55">

    <defs>
      <pattern id="woodpattern" patternUnits="userSpaceOnUse" width="100" height="50" patternTransform="rotate(90)">
        <image href="https://t4.ftcdn.net/jpg/03/01/86/13/360_F_301861323_tEKMCqC9DDuRKTkGR6cLz4x4mQ4dSlDC.jpg" x="0" y="-12" width="50" height="50" />
        <image href="https://t4.ftcdn.net/jpg/03/01/86/13/360_F_301861323_tEKMCqC9DDuRKTkGR6cLz4x4mQ4dSlDC.jpg" x="0" y="12" width="50" height="50" />
      </pattern>
    </defs>

    <!-- <rect width = "57" height = "50" fill = "red" style = "transform:translate(-29px, -3px)"></rect> -->
    <rect width = "50" height = "47" fill = "#fff8dc" style = "transform:translate(-25px, 0px)"></rect>

    {#each paints as line}
      <path
        d = "{lineGenerator(courtnested[line])}"
        fill = "transparent"
        stroke = "#17408b"
        stroke-width = 0.15
      />
    {/each}

    <g>
    {#if shotsquery.length>0}
      {#each shotnumbers as dot}
    {#key shotsquery[dot]}
      <circle
        transition:fade={{delay:100, duration:500}}
        cx="{shotsquery[dot].loc_x}"
        cy="{shotsquery[dot].loc_y}"
        r=0.3
        stroke="{colorScale(shotsquery[dot].event_type)}"
        fill="transparent"
        on:mouseenter={calculateTooltip(shotsquery[dot])}
        on:mouseleave={emptyTooltip(dot)}
        stroke-width="{widthScale(shotsquery[dot].event_type)}"
      />
      {/key}
      {/each}
    {/if}
    </g>
  </ZoomSvg>

    <div class = "bleachers" id="bleachers-right">
    <div class = "metrics-box"></div>
    </div>

    <!-- style="transform-origin:left;
         transform: translate(calc(-50% + {xPos}px + {xPosZoom}px),
                              calc(-100% + {yPos}px + {yPosZoom}px - 8px));  -->

  <div class = "{scaleZoom==1 ? 'tooltip-moving' : 'tooltip-static' }"
  style="transform-origin:left;
         transform: translate(calc(-50% + {xPosZoom}px),
                              calc(-100% + {yPosZoom}px - 8px));
         opacity:{opac};">
    <div class="tooltip-player">
          {@html tooltipBody}
    </div>
    <hr>
    <div class="tooltip-league-average">
      League average shot percentage from this position: <span id="league-average">{Math.trunc(league_av*100)}%</span>
    </div>
  </div>

</div>

<style>
  .wrapper {
    position: relative;
    display: flex;
    flex-direction: column;
    height: 80vh;
    margin: auto;
    flex-wrap: wrap;
    gap: 20px;
    align-content: space-around;
    justify-content: baseline;
  }

  .bleachers {
    height: 90%;
    width: 160px;
    border: 1px solid hsl(349, 92%, 41%);
    background-color: #f4cdd4;
    opacity: 0.8;
    background-size: 20px 20px;
  }

  #bleachers-left {
    background-image: repeating-linear-gradient(
      to right,
      black,
      darkgrey 1px,
      lightgrey 3px,
      hsl(349, 92%, 92%) 1px,
      hsl(349, 92%, 92%)
    );
    box-shadow: rgba(0, 0, 0, 0.05) 0px 0px 0px 1px,
      rgb(209, 213, 219) 0px 0px 0px 1px inset;
  }

  #bleachers-right {
    background-image: repeating-linear-gradient(
      to left,
      black,
      darkgrey 1px,
      lightgrey 3px,
      hsl(349, 92%, 92%) 1px,
      hsl(349, 92%, 92%)
    );
    box-shadow: rgba(0, 0, 0, 0.05) 0px 0px 0px 1px,
      rgb(209, 213, 219) 0px 0px 0px 1px inset;
  }

  .metrics-box {
    width: 135px;
    height: 150px;
    border: 1px solid hsla(349, 92%, 41%, 0.7);
    background-color: hsla(349, 92%, 70%, 0.25);
    margin: 10px;
    margin-top: 20px;
    backdrop-filter: blur(6px);
    box-shadow: rgba(9, 30, 66, 0.25) 0px 1px 1px,
      rgba(9, 30, 66, 0.13) 0px 0px 1px 1px;
    background-color: hsl(219, 72%, 60%, 0.3);
  }

  .tooltip-static {
    opacity: 0;
    position: absolute;
    top: 780px;
    left: calc(50% - 70px);
    padding: 0.6em 1em;
    background: #fff;
    text-align: center;
    line-height: 1.2rem;
    font-size: 0.9rem;
    border: 1px solid #17408b;
    z-index: 10;
    transition: all 0.1s ease-out;
    pointer-events: none;
    max-width: 320px;
    border-radius: 0 0 4px 0;
  }

  .tooltip-static:before {
    content: "";
    position: absolute;
    top: -16px;
    left: 50%;
    width: 14px;
    height: 14px;
    background: white;
    border: 1px solid #17408b;
    border-top-color: transparent;
    border-left-color: transparent;
    transform: translate(-50%, 50%) rotate(-135deg);
    transform-origin: center center;
    z-index: 10;
  }

  .tooltip-moving {
    opacity: 0;
    position: absolute;
    top: -14px;
    left: 0;
    padding: 0.6em 1em;
    background: #fff;
    text-align: center;
    line-height: 1.2rem;
    font-size: 0.9rem;
    border: 1px solid #17408b;
    z-index: 10;
    transition: all 0.1s ease-out;
    pointer-events: none;
    max-width: 320px;
  }

  .tooltip-moving:before {
    content: "";
    position: absolute;
    bottom: 0;
    left: 50%;
    width: 12px;
    height: 12px;
    background: white;
    border: 1px solid #17408b;
    border-top-color: transparent;
    border-left-color: transparent;
    transform: translate(-50%, 50%) rotate(45deg);
    transform-origin: center center;
    z-index: 10;
  }

  .tooltip-league-average {
    margin-top: 10px;
  }

  #league-average {
    font-weight: 700;
    color: #17408b;
  }

  circle:hover {
    stroke-width: 0.2;
    r: 0.5;
  }
</style>


