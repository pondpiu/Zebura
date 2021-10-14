<script lang="ts">
  import zebas from "./assets/zebas.jpg";
  import Result from "./page/Result.svelte";
  import Vote from "./page/Vote.svelte";
  import { auth, signInAnonymously, firestore } from "./firebase";
  import { authState } from "rxfire/auth";
  import {
    collection,
    doc,
    orderBy,
    query,
    setDoc,
    where,
  } from "firebase/firestore";
  import { collectionData, docData } from "rxfire/firestore";
  import {
    combineLatest,
    filter,
    map,
    pairwise,
    share,
    startWith,
    switchMap,
    tap,
  } from "rxjs";
  import Participant from "./lib/Participant.svelte";

  let user;
  let userData;

  const userData$ = authState(auth).pipe(
    filter((u) => u !== null),
    tap((u) => {
      user = u;
    }),
    switchMap((u, i) => {
      const userRef = doc(firestore, "User", u?.uid);
      return docData(userRef);
    }),
    tap((ud) => {
      userData = ud;
    })
  );
  let roomData;
  let scores = [];

  const roomRef = doc(firestore, "Room", "1");
  const roomData$ = docData(roomRef).pipe(
    tap((rd) => {
      roomData = rd;
    }),
    share()
  );
  roomData$
    .pipe(
      switchMap((rd, i) => {
        const scoreCardRef = query(
          collection(firestore, "ScoreCard"),
          where("voteId", "==", rd.voteId),
          orderBy("value")
        );
        return collectionData(scoreCardRef, { idField: "uid" });
      })
    )
    .subscribe((s) => {
      scores = s;
    });

  const newVoteIdRoomData$ = roomData$.pipe(
    startWith(undefined),
    pairwise(),
    filter((data) => {
      const [previousRoomData, currentRoomData] = data;
      return previousRoomData?.voteId !== currentRoomData?.voteId;
    }),
    map((data) => {
      const [_, currentRoomData] = data;
      return currentRoomData;
    })
  );

  combineLatest([userData$, newVoteIdRoomData$]).subscribe(() => {
    setRole();
  });

  function login() {
    signInAnonymously(auth).then((uc) => {
      const name = prompt("Please enter your name");
      const uid = uc.user.uid;
      setDoc(doc(firestore, "User", uid), {
        name,
      });
    });
  }

  function rename() {
    const name = prompt("Please enter your name");
    const uid = user.uid;
    setDoc(doc(firestore, "User", uid), {
      name,
    });
  }

  let isObserver = false;
  function toggleResult() {
    isObserver = !isObserver;
    setRole();
  }

  function setRole() {
    const pointRef = doc(firestore, "ScoreCard", user.uid);
    setDoc(pointRef, {
      username: userData.name,
      value: isObserver ? "observer" : null,
      voteId: roomData.voteId,
    });
  }
</script>

<main>
  <img src={zebas} alt="Zebas" />
  <section>
    {#if user && userData}
      <div class="grid-container">
        <div class="mainBox">
          {#if roomData}
            {#if isObserver || roomData.status === "result"}
              <Result roomStatus={roomData.status} {scores} />
            {:else}
              <Vote voteId={roomData.voteId} {userData} uid={user.uid} />
            {/if}
          {:else}
            Loading
          {/if}
        </div>
        <div class="sideBox">
          <Participant {scores} />

          <div class="sideBottom">
            <button on:click={rename}>Rename</button>
          </div>
        </div>
      </div>
      <button on:click={toggleResult}
        >{isObserver ? "Join Voting" : "Become observer"}</button
      >
    {:else}
      <button on:click={login}> Signin </button>
    {/if}
  </section>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  .mainBox {
    height: auto;
    min-height: 20em;
  }

  .sideBottom {
    margin-top: 20em;
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  img {
    height: 16rem;
    width: 16rem;
  }

  .grid-container {
    display: grid;
    grid-template-columns: 4fr 1fr;
    grid-gap: 20px;
  }
</style>
