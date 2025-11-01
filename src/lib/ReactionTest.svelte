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
        lights = [...lights];
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
  <!-- Background particles -->
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
      ← BACK
    </button>
  </div>
</main>

<style>
:global(body) {
  margin: 0;
  overflow: hidden;
  font-family: 'Orbitron', sans-serif;
}

/* Main gradient background with animation */
main {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  background-size: 400% 400%;
  animation: gradientShift 20s ease infinite;
  color: white;
  text-align: center;
  cursor: pointer;
  position: relative;
}

main.jumpstart {
  background: linear-gradient(135deg, #4b0000, #ff0000);
}

@keyframes gradientShift {
  0%{background-position:0% 50%;}
  50%{background-position:100% 50%;}
  100%{background-position:0% 50%;}
}

/* Background particles */
.background-particles {
  position: absolute;
  top:0;
  left:0;
  width:100%;
  height:100%;
  pointer-events: none;
  z-index:0;
}
.particle {
  position: absolute;
  border-radius: 50%;
  background: rgba(255,255,255,0.1);
  animation: floatParticle linear infinite;
}
@keyframes floatParticle {
  0% { transform: translateY(0) translateX(0); }
  50% { transform: translateY(-10px) translateX(5px); }
  100% { transform: translateY(0) translateX(0); }
}

/* Container and text */
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  z-index: 10;
}

h1 {
  font-size: 3rem;
  font-weight: 900;
  color: #f39c12;
  text-shadow: 0 0 20px #f39c12, 0 0 40px #e74c3c;
}

.instruction {
  font-size: 1.3rem;
  color: #ccc;
  text-shadow: 0 0 5px #000;
}

.lights {
  display: flex;
  gap: 1rem;
}

.light {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: rgba(255,0,0,0.2);
  border: 3px solid #400;
  box-shadow: inset 0 0 10px rgba(0,0,0,0.5);
  transition: all 0.3s ease, transform 0.2s ease;
}
.light.on {
  background: #e74c3c;
  border-color: #f00;
  box-shadow: 0 0 25px #e74c3c, 0 0 50px #e74c3c, 0 0 80px #f39c12;
  transform: scale(1.1);
}

.best {
  font-size: 1.3rem;
  font-weight: bold;
  color: #2ecc71;
  text-shadow: 0 0 15px #2ecc71;
}

.back-btn { 
  position: absolute; top: 20px; left: 20px; padding: 0.8rem 1.6rem; 
  background: rgba(0,0,0,0.6); color: white; border: 1px solid #e74c3c; 
  border-radius: 10px; font-weight: bold; cursor: pointer; transition: all 0.3s ease; z-index: 100; 
}
.back-btn:hover { 
  background: #e74c3c; transform: translateY(-3px); box-shadow: 0 0 20px #e74c3c; 
}

</style>
