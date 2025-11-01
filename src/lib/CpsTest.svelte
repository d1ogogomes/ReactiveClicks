<script>
  import { createEventDispatcher, onMount, onDestroy } from "svelte";
  import FireBar from "./UI/FireBar.svelte";

  const dispatch = createEventDispatcher();

  let count = 0;
  let timeLeft = 10;
  let timerRunning = false;
  let cps = 0;
  let currentCPS = 0;
  let interval;
  let intensity = 0;
  let stars = [];
  let started = false;

  // Start the 10-second test
  function start() {
    count = 0;
    timeLeft = 10;
    cps = 0;
    currentCPS = 0;
    intensity = 0;
    timerRunning = true;
    started = true;

    interval = setInterval(() => {
      timeLeft -= 0.1;
      if (timeLeft <= 0) {
        clearInterval(interval);
        timerRunning = false;
        cps = (count / 10).toFixed(2);
        currentCPS = 0;
      } else {
        currentCPS = (count / (10 - timeLeft)).toFixed(2);
      }

      if (intensity > 0) intensity -= 0.5;
    }, 100);
  }

  // Handle click — starts or counts
  function clickBtn() {
    if (!started) {
      start();
      return;
    }
    if (!timerRunning) return;
    count += 1;
    intensity += 5;
    if (intensity > 100) intensity = 100;
  }

  // Reset on click anywhere after test
  function reset() {
    if (!timerRunning && cps > 0) {
      started = false;
      count = 0;
      timeLeft = 10;
      cps = 0;
      currentCPS = 0;
    }
  }

  // Generate golden stars
  onMount(() => {
    stars = Array.from({ length: 80 }, () => ({
      x: Math.random() * 100,
      y: Math.random() * 100,
      size: Math.random() * 2 + 1,
      duration: Math.random() * 3 + 2,
      delay: Math.random() * 5
    }));
  });

  onDestroy(() => {
    clearInterval(interval);
  });
</script>

<svelte:window on:click={reset} />

<main class="cps-test">
  <!-- Background stars -->
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
  
  <button class="back-btn" on:click={() => dispatch("back")}>
    ← BACK
  </button>

  <!-- Title -->
  <div class="title">
    <h1>CPS<span>TEST</span></h1>
  </div>

  <!-- HUD -->
  <div class="hud">
    <div class="stat">
      <span class="label">Time</span>
      <span class="value">{timeLeft.toFixed(1)}s</span>
    </div>
    <div class="stat">
      <span class="label">Clicks</span>
      <span class="value">{count}</span>
    </div>
    <div class="stat">
      <span class="label">CPS</span>
      <span class="value">{timerRunning ? currentCPS : (started ? cps : '0.00')}</span>
    </div>
  </div>

  <!-- Main button -->
  <div class="button-wrapper">
    <button 
      on:click={clickBtn} 
      class="click-btn"
      class:disabled={!timerRunning && started}
    >
      {#if !started}
        CLICK TO START
      {:else}
        FIRE
      {/if}
    </button>
  </div>

  <!-- Result -->
  {#if !timerRunning && cps > 0}
    <div class="result">
      <p>Your CPS: <strong>{cps}</strong></p>
      <p class="hint">Click anywhere to restart</p>
    </div>
  {/if}

  <!-- FireBar -->
  {#if timerRunning}
    <div class="bar-container">
      <FireBar {intensity} />
    </div>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    overflow: hidden;
    background: #0a0a0a;
    font-family: 'Orbitron', 'Arial', sans-serif;
  }

  .cps-test {
    position: relative;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    background: radial-gradient(circle at center, #1a1a2e, #0f0f1e);
    overflow: hidden;
    color: white;
    text-align: center;
    cursor: crosshair;
    padding-top: 8vh; 
    gap: 3.5rem; 
  }

  /* STARS */
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

  /* BACK BUTTON */
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

  /* TITLE */
  .title {
    margin-top: 1vh;
    margin-bottom: 2rem;
  }
  h1 {
    font-size: 3.2rem;
    font-weight: 900;
    background: linear-gradient(45deg, #f39c12, #e74c3c, #f1c40f);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 30px rgba(241, 196, 15, 0.5);
    margin: 0;
    z-index: 10;
  }
  h1 span {
    display: block;
    font-size: 1.4rem;
    letter-spacing: 0.4em;
    color: #aaa;
    margin-top: -0.6rem;
  }

  /* HUD */
  .hud {
    display: flex;
    gap: 2.5rem;
    flex-wrap: wrap;
    justify-content: center;
    margin-bottom: 2.5rem;
  }
  .stat {
    background: rgba(0,0,0,0.6);
    padding: 1.2rem 2rem;
    border-radius: 15px;
    border: 2px solid #e67e22;
    text-align: center;
    backdrop-filter: blur(10px);
    min-width: 130px;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
  }
  .stat:hover {
    transform: scale(1.05);
    box-shadow: 0 0 25px #f39c12;
  }
  .label {
    font-size: 0.95rem;
    color: #f1c40f;
    text-shadow: 0 0 6px #f1c40f;
    font-weight: bold;
  }
  .value {
    font-size: 2.1rem;
    font-weight: 900;
    color: #e74c3c;
    text-shadow: 0 0 16px #e74c3c;
  }

  /* BUTTON — FIXED SPOT */
  .button-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 260px;
    margin-bottom: 2rem;
  }

  .click-btn {
    width: 220px;
    height: 220px;
    font-size: 1.8rem;
    border-radius: 50%;
    background: radial-gradient(circle, #e74c3c, #f39c12);
    color: white;
    box-shadow: 0 0 30px #f1c40f, 0 0 60px #e74c3c;
    border: none;
    cursor: pointer;
    text-transform: uppercase;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 1rem;
    line-height: 1.2;
    transition: all 0.2s ease;
  }
  .click-btn:active:not(:disabled) {
    transform: scale(0.9);
    box-shadow: 0 0 50px #f1c40f, 0 0 70px #e74c3c;
  }
  .click-btn:disabled {
    opacity: 0.3;
    cursor: not-allowed;
    box-shadow: 0 0 10px #666;
  }

  .result {
    margin-top: 1rem;
  }
  .result p {
    font-size: 2rem;
    margin: 0 0 0.5rem;
    text-shadow: 0 0 12px #f39c12;
  }
  .result strong {
    color: #f1c40f;
    text-shadow: 0 0 18px #f1c40f;
  }
  .hint {
    font-size: 1.1rem;
    color: #f1c40f;
    opacity: 0.85;
    cursor: pointer;
    text-shadow: 0 0 8px #f1c40f;
  }

  /* FIREBAR */
  .bar-container {
    width: 80%;
    max-width: 700px;
    margin: 0 auto 3rem;
    display: flex;
    justify-content: center;
  }
</style>
