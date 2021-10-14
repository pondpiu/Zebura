<script lang="ts">
  import { quintOut } from "svelte/easing";
  import { crossfade, fly } from "svelte/transition";
  import { flip } from "svelte/animate";

  import { firestore } from "../firebase";
  import { doc, setDoc } from "firebase/firestore";
  import ScoreCard from "../lib/ScoreCard.svelte";

  import countBy from "lodash/countBy";

  const [send, receive] = crossfade({
    duration: (d) => Math.sqrt(d * 5000),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
      };
    },
  });

  export let roomStatus;
  export let scores: any[];
  $: filteredScores = scores.filter((s) => s.value > 0);
  $: counts = countBy(filteredScores, "value");
  function toggleResult() {
    const roomRef = doc(firestore, "Room", "1");
    const newStatus = roomStatus === "voting" ? "result" : "voting";
    setDoc(
      roomRef,
      {
        status: newStatus,
      },
      { merge: true }
    );
  }

  function startNewVote() {
    const roomRef = doc(firestore, "Room", "1");
    setDoc(roomRef, {
      status: "voting",
      voteId: Math.floor(Math.random() * 100000),
    });
  }
</script>

<div>
  <h1>Vote Result</h1>
  {#each filteredScores as score (score.uid)}
    <div
      in:receive={{ key: score.uid }}
      out:send|local={{ key: score.uid }}
      animate:flip
    >
      <ScoreCard {score} hidden={roomStatus === "voting"} />
    </div>
  {/each}
  <div>
    {#each Object.entries(counts) as [point, amount] (point)}
      <span
        in:receive={{ key: point }}
        out:send|local={{ key: point }}
        animate:flip
      >
        [{roomStatus === "voting" ? "?" : point}] :
        {#key amount}
          <span style="display: inline-block" in:fly={{ y: -20 }}>
            {amount}
          </span>
        {/key}
      </span>
    {/each}
  </div>
  <button on:click={toggleResult}
    >{roomStatus === "voting" ? "Show Result" : "Resume Voting"}</button
  >
  {#if roomStatus === "result"}
    <button on:click={startNewVote}> new vote</button>
  {/if}
</div>

<style>
  span {
    padding: 1em;
  }
</style>
