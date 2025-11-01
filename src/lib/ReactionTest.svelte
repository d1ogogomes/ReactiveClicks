<script>
  import { onMount, onDestroy, createEventDispatcher } from 'svelte';
  import { fly, fade } from 'svelte/transition';
  import { quintOut } from 'svelte/easing';

  const dispatch = createEventDispatcher();

  let lights = [false, false, false, false, false];
  let phase = 'idle';
  let canClick = false;
  let startTime = 0;
  let reactionTime = 0;
  let bestTime = Number(localStorage.getItem('f1-best')) || Infinity;
  let jumpstart = false;
  let timeoutId;
  let stars = [];

  const beepSound = new Audio(
    'data:audio/wav;base64,UklGRnoGAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQoGAACBhYqFbF1fdJivrJBhNjVgodDbq2EcBQAAAP//AAAAP/7k'
  );

  function startSequence() {
    if (phase !== 'idle') return;
    phase = 'lighting'; canClick = false; reactionTime = 0; jumpstart = false;
    lights = [false, false, false, false, false]; let i = 0;

    function lightNext() {
      if (i < 5) {
        lights[i] = true; lights = [...lights];
        beepSound.currentTime = 0; beepSound.play().catch(() => {});
        i++;
        timeoutId = setTimeout(lightNext, 600 + Math.random() * 400);
      } else {
        timeoutId = setTimeout(() => {
          lights.fill(false); lights = [...lights];
          phase = 'ready'; canClick = true; startTime = performance.now();
        }, 1200 + Math.random() * 1000);
      }
    }
    lightNext();
  }

  function handleClick() {
    if (phase === 'idle') { startSequence(); return; }
    if (!canClick && phase === 'lighting') { phase = 'done'; jumpstart = true; reactionTime = -1; setTimeout(reset, 1500); return; }
    if (!canClick) return;

    reactionTime = performance.now() - startTime;
    canClick = false; phase = 'done';
    if (reactionTime < bestTime) { bestTime = reactionTime; localStorage.setItem('f1-best', bestTime); }
    setTimeout(reset, 2000);
  }

  function reset() { phase = 'idle'; clearTimeout(timeoutId); }

  onMount(() => {
    document.addEventListener('click', handleClick);

    stars = Array.from({ length: 80 }, () => ({
      x: Math.random() * 100,
      y: Math.random() * 100,
      size: Math.random() * 2 + 1,
      duration: Math.random() * 3 + 2,
      delay: Math.random() * 5
    }));
  });

  onDestroy(() => {
    document.removeEventListener('click', handleClick);
    clearTimeout(timeoutId);
  });
</script>

<main class:jumpstart={jumpstart} in:fly={{ y: 50, duration: 800, easing: quintOut }}>
  <!-- ESTRELAS -->
  {#each stars as star (star.x + star.y)}
    <div
      class="particle"
      style="
        left:{star.x}%;
        top:{star.y}%;
        width:{star.size}px;
        height:{star.size}px;
        animation: floatParticle {star.duration}s infinite {star.delay}s;
      "
    ></div>
  {/each}

  <!-- BACK BUTTON -->
  <button class="back-btn" on:click={() => dispatch("back")}>
    ← BACK
  </button>


  <div class="title">
    <h1>F1<span>REACTION</span></h1>
  </div>


  <p class="instruction">
    {#if phase === 'idle'}Click to start!{/if}
    {#if phase === 'lighting'}Get ready...{/if}
    {#if phase === 'ready'}GO!{/if}
    {#if jumpstart}FALSE START!{/if}
    {#if phase === 'done' && !jumpstart}{reactionTime.toFixed(0)} ms{/if}
  </p>


  <div class="lights-container">
    <div class="lights">
      {#each lights as on, i}
        <div class="light" class:on={on} in:fly={{ x: -50, delay: i * 80 }}></div>
      {/each}
    </div>
  </div>

  <!-- BEST NO FUNDO -->
  <p class="best">
    Best: {bestTime === Infinity ? '---' : bestTime.toFixed(0)} ms
  </p>
</main>

<style>
  :global(body) {
    margin: 0;
    overflow: hidden;
    background: #0a0a0a;
    font-family: 'Orbitron', 'Arial', sans-serif;
  }

  main {
    position: relative;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: space-between; /* centralização vertical mais equilibrada */
    background: radial-gradient(circle at center, #1a1a2e, #0f0f1e);
    overflow: hidden;
    color: white;
    cursor: pointer;
    padding: 8vh 0 8vh 0; /* margem superior e inferior */
  }

  main.jumpstart {
    background: radial-gradient(circle at center, #3a0000, #1a0000);
  }

  .particle {
    position: absolute;
    background: #f39c12;
    border-radius: 50%;
    box-shadow: 0 0 10px #f39c12, 0 0 20px #f39c12;
    pointer-events: none;
    opacity: 0.7;
  }

  @keyframes floatParticle {
    0% { transform: translateY(0) translateX(0); }
    50% { transform: translateY(-10px) translateX(5px); }
    100% { transform: translateY(0) translateX(0); }
  }

  .back-btn {
    position: fixed;
    top: 20px;
    left: 20px;
    padding: 0.8rem 1.6rem;
    background: rgba(0,0,0,0.6);
    color: white;
    border: 1px solid #e74c3c;
    border-radius: 10px;
    font-weight: bold;
    cursor: pointer;
    transition: all 0.3s ease;
    z-index: 1000;
    font-size: 0.9rem;
  }
  .back-btn:hover {
    background: #e74c3c;
    transform: translateY(-3px);
    box-shadow: 0 0 20px #e74c3c;
  }

  .title {
    text-align: center;
  }

  h1 {
    font-size: 3.2rem;
    font-weight: 900;
    background: linear-gradient(45deg, #f39c12, #e74c3c, #f1c40f);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 30px rgba(241, 196, 15, 0.5);
    margin: 0;
  }
  h1 span {
    display: block;
    font-size: 1.4rem;
    letter-spacing: 0.4em;
    color: #aaa;
    margin-top: -0.6rem;
  }

  .instruction {
    font-size: 1.6rem;
    font-weight: bold;
    color: #f1c40f;
    text-shadow: 0 0 15px #f1c40f;
    margin: 2rem 0 1rem 0;
    z-index: 10;
  }

  .lights-container {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    padding-top: 2vh;
  }

  .lights {
    display: flex;
    gap: 2rem;
    justify-content: center;
    align-items: center;
  }

  .light {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    background: rgba(255, 0, 0, 0.2);
    border: 3px solid #400;
    box-shadow: inset 0 0 10px rgba(0,0,0,0.5);
    transition: all 0.3s ease;
  }
  .light.on {
    background: #e74c3c;
    border-color: #f00;
    box-shadow: 0 0 25px #e74c3c, 0 0 50px #e74c3c, 0 0 80px #f39c12;
    transform: scale(1.1);
  }

  .best {
    font-size: 1.5rem;
    color: #2ecc71;
    text-shadow: 0 0 15px #2ecc71;
    font-weight: bold;
    z-index: 10;
  }

  @media (max-width: 600px) {
    .title { margin-top: 5vh; }
    h1 { font-size: 2.5rem; }
    h1 span { font-size: 1.1rem; }
    .instruction { font-size: 1.3rem; margin: 2rem 0 1rem 0; }
    .light { width: 60px; height: 60px; }
    .lights { gap: 1.2rem; }
  }
</style>
