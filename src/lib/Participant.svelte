<script lang="ts">
  import { flip } from "svelte/animate";

  export let scores: any[];
  $: observerScores = scores.filter((s) => s.value === "observer");
  $: idleScores = scores.filter((s) => !s.value);
  $: activeScores = scores
    .filter((s) => s.value > 0)
    .sort((a, b) => (a.username > b.username && 1) || -1);
  $: sortedScores = [...idleScores, ...activeScores, ...observerScores];
</script>

<h2>Participant</h2>
{#each sortedScores as score (score.uid)}
  <div animate:flip>
    {score.username} : {score.value === "observer"
      ? "👀"
      : score.value > 0
      ? "✅"
      : "💤"}
  </div>
{/each}

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }
</style>
