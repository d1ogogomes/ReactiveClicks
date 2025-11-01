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

  function clickBtn() {
    if (!timerRunning) return;
    count += 1;
    intensity += 5;
    if (intensity > 100) intensity = 100;
  }
</script>

<main>
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
      <button on:click={start} class="start-btn">Try Again</button>
    </div>
  {:else if !timerRunning}
    <button on:click={start} class="start-btn">Start</button>
  {/if}

  <button class="click-btn" on:click={clickBtn} disabled={!timerRunning}>
    🔥 Click Me!
  </button>

  <FireBar {intensity} />

  <button class="back-btn" on:click={() => dispatch("back")}>← Back to Menu</button>
</main>

<style>
:global(body) { margin: 0; overflow: hidden; background: #111; font-family: 'Orbitron', sans-serif; color: white; }

main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  gap: 2rem;
  text-align: center;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  background-size: 400% 400%;
  animation: gradientShift 20s ease infinite;
}

@keyframes gradientShift {
  0%{background-position:0% 50%;}
  50%{background-position:100% 50%;}
  100%{background-position:0% 50%;}
}

h1 {
  font-size: 3rem;
  text-transform: uppercase;
  color: #f39c12;
  text-shadow: 0 0 15px #f39c12, 0 0 30px #f39c12;
}

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

.click-btn {
  width: 200px;
  height: 200px;
  font-size: 1.8rem;
  border-radius: 50%;
  background: linear-gradient(45deg, #e74c3c, #f39c12);
  color: white;
  border: none;
  cursor: pointer;
  box-shadow: 0 0 25px #f39c12, 0 0 40px rgba(231,76,60,0.5);
  transition: transform 0.1s ease, box-shadow 0.1s ease;
}
.click-btn:active {
  transform: scale(0.9);
  box-shadow: 0 0 50px #f1c40f, 0 0 70px rgba(231,76,60,0.5);
}

.start-btn {
  padding: 1rem 2rem;
  font-size: 1.6rem;
  border-radius: 12px;
  background: linear-gradient(45deg, #f39c12, #e74c3c);
  border: none;
  color: white;
  cursor: pointer;
  box-shadow: 0 5px 15px rgba(231,76,60,0.6);
  transition: transform 0.2s ease, box-shadow 0.2s ease, background 0.3s ease;
}
.start-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 8px 25px rgba(231,76,60,0.8);
  background: linear-gradient(45deg, #f1c40f, #e74c3c);
}
.start-btn:active {
  transform: scale(0.95);
  box-shadow: 0 3px 12px rgba(231,76,60,0.6);
}

.result p {
  font-size: 1.8rem;
  margin-bottom: 1rem;
  text-shadow: 0 0 10px #f39c12;
}

.back-btn {
  padding: 0.6rem 1.2rem;
  border-radius: 8px;
  border: none;
  background: #e67e22;
  color: white;
  cursor: pointer;
  font-weight: bold;
  transition: 0.2s;
}
.back-btn:hover {
  transform: scale(1.05);
  box-shadow: 0 0 15px #f39c12;
}
</style>
