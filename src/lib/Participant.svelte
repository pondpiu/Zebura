<script lang="ts">
  export let scores: any[];
  $: observerScores = scores.filter((s) => s.value === "observer");
  $: idleScores = scores.filter((s) => !s.value);
  $: activeScores = scores
    .filter((s) => s.value > 0)
    .sort((a, b) => (a.username > b.username && 1) || -1);
</script>

<h2>Participant</h2>
{#each idleScores as score}
  <div>
    {score.username} : 💤
  </div>
{/each}
{#each activeScores as score}
  <div>
    {score.username} : ✅
  </div>
{/each}
{#each observerScores as score}
  <div>
    {score.username} : 👀
  </div>
{/each}

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }
</style>
