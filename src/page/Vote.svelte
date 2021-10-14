<script lang="ts">
  import { fade, fly } from "svelte/transition";

  import { doc, setDoc } from "firebase/firestore";

  import ScoreButton from "../lib/ScoreButton.svelte";
  import { firestore } from "../firebase";

  export let voteId;
  export let userData;
  export let uid;

  const points = [1, 2, 3, 5, 8];

  function vote(event) {
    const pointRef = doc(firestore, "ScoreCard", uid);
    const { point } = event.detail;
    setDoc(
      pointRef,
      {
        value: point,
        voteId,
      },
      { merge: true }
    );
  }
</script>

<div in:fade>
  <h2>
    Make your vote {userData.name}
  </h2>
  <div in:fly={{ y: 200, duration: 2000 }}>
    {#each points as point}
      <ScoreButton {point} on:click={vote} />
    {/each}
  </div>
</div>

<style>
</style>
