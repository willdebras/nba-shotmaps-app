<script>
  import matchups from "./public/matchups.json";
  import * as lodash from "lodash";
  import Shotmap from "./Shotmap.svelte";

  const team1_array = matchups.map(a => a["team1_name"]);
  const team1_select = lodash.uniq(team1_array);

  $: team2_array = matchups
    .filter(d => d.team2_name.includes(team1))
    .map(a => a["team1_name"]);
  $: team2_select = lodash.uniq(team2_array);

  $: date_array = matchups
    .filter(d => d.team1_name.includes(team1) && d.team2_name.includes(team2))
    .map(a => a["game_date"]);
  $: date_select = lodash.uniq(date_array);

  $: gameid = matchups
    .filter(
      d =>
        d.team1_name.includes(team1) &&
        d.team2_name.includes(team2) &&
        d.game_date.includes(date)
    )
    .map(a => a["game_id"]);

  export let team1;
  export let team2;
  export let date;
</script>

<span>Shotmaps for the</span> 
<select bind:value={team1} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each team1_select as team}
    <option value={team}>{team}</option>
  {/each}
</select>
<span>vs. </span> 
<select bind:value={team2} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each team2_select as team, i}
    <option value={team}>{team}</option>
  {/each}
</select>
<span>on </span> 
<select bind:value={date} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each date_select as date}
    <option value={date}>{date}</option>
  {/each}
</select>

<Shotmap gameinfo={gameid} team1={team1} team2={team2}></Shotmap>

<style>
  span {
    font-size: 1.1rem;
    margin: 4px;
  }
  select {
    border: 0;
    border-bottom: 2px solid #c9082a;
    color: #c9082a;
    font-size: 1.2rem;
    font-weight: 400;
    background-color: #f4f5f2;
  }
  select:focus {
    outline: none;
  }
</style>