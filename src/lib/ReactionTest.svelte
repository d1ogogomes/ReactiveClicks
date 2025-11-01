<script>
  import { onMount, onDestroy, createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let lights = [false, false, false, false, false];
  let phase = 'idle'; // idle, lighting, ready, done
  let canClick = false;
  let startTime = 0;
  let reactionTime = 0;
  let bestTime = Number(localStorage.getItem('f1-best')) || Infinity;
  let jumpstart = false;
  let timeoutId;

  const beepSound = new Audio('data:audio/wav;base64,UklGRnoGAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQoGAACBhYqFbF1fdJivrJBhNjVgodDbq2EcBQAAAP//AAAAP/7k');

  function startSequence() {
    if (phase !== 'idle') return;

    phase = 'lighting';
    canClick = false;
    reactionTime = 0;
    jumpstart = false;
    lights = [false, false, false, false, false];
    let index = 0;

    function lightNext() {
      if (index < 5) {
        lights[index] = true;
        lights = [...lights]; // trigger Svelte update
        beepSound.currentTime = 0;
        beepSound.play().catch(() => {});
        index++;
        timeoutId = setTimeout(lightNext, 700 + Math.random() * 600);
      } else {
        timeoutId = setTimeout(() => {
          lights.fill(false);
          lights = [...lights];
          phase = 'ready';
          canClick = true;
          startTime = performance.now();
        }, 1000 + Math.random() * 2000);
      }
    }

    lightNext();
  }

  function handleClick() {
    if (phase === 'idle') {
      startSequence();
      return;
    }

    if (!canClick) {
      if (phase === 'lighting') {
        phase = 'done';
        jumpstart = true; // clicked too early
        reactionTime = -1;
        setTimeout(reset, 1500);
      }
      return;
    }

    reactionTime = performance.now() - startTime;
    canClick = false;
    phase = 'done';
    jumpstart = false;

    if (reactionTime < bestTime) {
      bestTime = reactionTime;
      localStorage.setItem('f1-best', bestTime);
    }

    setTimeout(reset, 2000);
  }

  function reset() {
    phase = 'idle';
    jumpstart = false;
    clearTimeout(timeoutId);
  }

  onMount(() => {
    document.addEventListener('click', handleClick);
  });

  onDestroy(() => {
    document.removeEventListener('click', handleClick);
    clearTimeout(timeoutId);
  });
</script>

<main class:jumpstart={jumpstart}>
  <div class="container">
    <h1>F1 Start Timer</h1>

    <p class="instruction">
      {#if phase === 'idle'}
        Click anywhere to start the sequence!
      {:else if phase === 'lighting'}
        Get ready...
      {:else if phase === 'ready'}
        GO!
      {:else if jumpstart}
        Jump Start! Too early!
      {:else}
        {reactionTime.toFixed(0)} ms
      {/if}
    </p>

    <div class="lights">
      {#each lights as on}
        <div class="light" class:on={on}></div>
      {/each}
    </div>

    <p class="best">
      Best: {bestTime === Infinity ? '00.000' : bestTime.toFixed(0)} ms
    </p>

    <button class="back-btn" on:click={() => dispatch("back")}>
      Back to Menu
    </button>
  </div>
</main>

<style>
  main {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: linear-gradient(135deg, #1f1c2c, #928dab);
    font-family: 'Orbitron', 'Arial', sans-serif;
    color: white;
    text-align: center;
    user-select: none;
    cursor: pointer;
    transition: background 0.3s;
  }

  main.jumpstart {
    background: linear-gradient(135deg, #4b0000, #ff0000);
  }

  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  h1 {
    font-size: 2.8rem;
    font-weight: 900;
    color: #e74c3c;
    text-shadow: 0 0 20px #e74c3c, 0 0 40px #e74c3c;
    margin-bottom: 1rem;
  }

  .instruction {
    font-size: 1.2rem;
    color: #ccc;
    margin-bottom: 2rem;
  }

  .lights {
    display: flex;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .light {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background: rgba(255,0,0,0.2);
    border: 3px solid #400;
    box-shadow: inset 0 0 10px rgba(0,0,0,0.5);
    transition: all 0.3s ease;
  }

  .light.on {
    background: #e74c3c;
    box-shadow: 0 0 20px #e74c3c, 0 0 40px #e74c3c, 0 0 60px #e74c3c;
    border-color: #f00;
  }

  .best {
    font-size: 1.2rem;
    font-weight: bold;
    color: #2ecc71;
    text-shadow: 0 0 10px #2ecc71;
    margin-bottom: 1rem;
  }

  button.back-btn {
    padding: 0.6rem 1.2rem;
    border-radius: 8px;
    border: none;
    background: #e67e22;
    color: white;
    cursor: pointer;
    font-weight: bold;
    transition: 0.2s;
  }

  button.back-btn:hover {
    background: #f39c12;
  }
</style>
