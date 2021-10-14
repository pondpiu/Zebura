<script lang="ts">
  export let score;
  export let hidden;
</script>

<div>
  {score.username}
  {hidden ? "???" : score.value}
</div>

<style>
</style>
