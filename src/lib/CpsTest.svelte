<script>
  import { createEventDispatcher } from "svelte";
  import FireBar from "./UI/FireBar.svelte";

  const dispatch = createEventDispatcher();

  let count = 0;
  let timeLeft = 10;
  let timerRunning = false;
  let cps = 0;
  let currentCPS = 0;
  let interval;
  let intensity = 0;

  function start() {
    if (timerRunning) return;

    count = 0;
    timeLeft = 10;
    cps = 0;
    currentCPS = 0;
    intensity = 0;
    timerRunning = true;

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

  function handleClick() {
    if (!timerRunning) return;
    count += 1;
    intensity += 5;
    if (intensity > 100) intensity = 100;
  }
</script>

<svelte:window on:click={handleClick} />

<main>
  <!-- Subtle animated background -->
  <div class="background-particles">
    {#each Array(80) as _, i}
      <div class="particle"
        style="
          top:{Math.random()*100}%;
          left:{Math.random()*100}%;
          width:{1 + Math.random()*2}px;
          height:{1 + Math.random()*2}px;
          animation-duration:{5 + Math.random()*10}s;
        ">
      </div>
    {/each}
  </div>

  <h1>CPS Test</h1>

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
      <span class="value">{timerRunning ? currentCPS : cps}</span>
    </div>
  </div>

  {#if !timerRunning && cps > 0}
    <div class="result">
      <p>Your CPS: {cps}</p>
      <button on:click={start} class="start-btn">TRY AGAIN</button>
    </div>
  {:else if !timerRunning}
    <button on:click={start} class="start-btn">START</button>
  {/if}

  <FireBar {intensity} />

  <button class="back-btn" on:click={() => dispatch("back")}>← BACK</button>
</main>

<style>
:global(body) {
  margin: 0;
  overflow: hidden;
  font-family: 'Orbitron', sans-serif;
  color: white;
}

main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  gap: 2rem;
  text-align: center;
  cursor: crosshair;
  position: relative;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  background-size: 400% 400%;
  animation: gradientShift 20s ease infinite;
}

/* Gradient background animation */
@keyframes gradientShift {
  0% { background-position:0% 50%; }
  50% { background-position:100% 50%; }
  100% { background-position:0% 50%; }
}

/* Floating background particles */
.background-particles {
  position: absolute;
  top:0; left:0;
  width:100%; height:100%;
  pointer-events:none;
  z-index:0;
}
.particle {
  position: absolute;
  border-radius: 50%;
  background: rgba(255,255,255,0.08);
  animation: floatParticle linear infinite;
}
@keyframes floatParticle {
  0% { transform: translateY(0) translateX(0);}
  50% { transform: translateY(-10px) translateX(5px);}
  100% { transform: translateY(0) translateX(0);}
}

/* Title */
h1 {
  font-size: 3rem;
  text-transform: uppercase;
  color: #f39c12;
  text-shadow: 0 0 15px #f39c12, 0 0 30px #f39c12;
  z-index: 10;
}

/* Stats HUD */
.hud {
  display: flex;
  gap: 2rem;
  z-index: 10;
}

.stat {
  background: rgba(0,0,0,0.6);
  padding: 1rem 1.8rem;
  border-radius: 15px;
  border: 2px solid #e67e22;
  text-align: center;
  backdrop-filter: blur(10px);
  min-width: 120px;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.stat:hover {
  transform: scale(1.05);
  box-shadow: 0 0 20px #f39c12;
}

.label {
  font-size: 0.9rem;
  color: #f1c40f;
  text-shadow: 0 0 5px #f1c40f;
  display: block;
  font-weight: bold;
}

.value {
  font-size: 2rem;
  font-weight: 900;
  color: #e74c3c;
  text-shadow: 0 0 15px #e74c3c;
}

/* Unified red-glow START button */
.start-btn {
  padding: 1rem 2.5rem;
  font-size: 1.4rem;
  border-radius: 12px;
  background: rgba(0,0,0,0.6);
  border: 2px solid #e74c3c;
  color: white;
  cursor: pointer;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 1px;
  box-shadow: 0 0 15px rgba(231,76,60,0.5);
  transition: all 0.3s ease;
  z-index: 10;
}
.start-btn:hover {
  background: #e74c3c;
  transform: translateY(-3px);
  box-shadow: 0 0 25px #e74c3c;
}
.start-btn:active {
  transform: scale(0.96);
  box-shadow: 0 0 10px #e74c3c;
}

/* Result text */
.result p {
  font-size: 1.8rem;
  margin-bottom: 1rem;
  text-shadow: 0 0 10px #f39c12;
  z-index: 10;
}

.back-btn { 
  position: absolute; 
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
  z-index: 100; 
}
.back-btn:hover { 
  background: #e74c3c; 
  transform: translateY(-3px); 
  box-shadow: 0 0 20px #e74c3c; 
}
</style>
