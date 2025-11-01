<script>
  import { createEventDispatcher, onMount, onDestroy } from "svelte";
  import { fade } from 'svelte/transition';

  const dispatch = createEventDispatcher();

  let score = 0;
  let lifes = 5;
  let x = 50;
  let y = 50;
  let timeLeft = 1;
  let interval;
  const targetSize = 80;

  let particles = [];
  let animationFrame;
  let bgIntensity = 0;

  // 🎵 Sons arcade leves (pequenos bips em base64)
  const hitSound = new Audio(
    "data:audio/wav;base64,UklGRoQAAABXQVZFZm10IBAAAAABAAEAESsAACJWAAACABAAZGF0YVQAAAB/////AAD///8AAP///wAA//8AAAD//wAA//8AAP///wAA//8AAP///wAA"
  );

  const missSound = new Audio(
    "data:audio/wav;base64,UklGRqIAAABXQVZFZm10IBAAAAABAAEAESsAACJWAAACABAAZGF0YVoAAACAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICA="
  );

  function randomPosition() {
    x = Math.random() * 70 + 15;
    y = Math.random() * 60 + 20;
    timeLeft = 1;
  }

  function hit(e) {
    e.stopPropagation();
    score += 1;
    bgIntensity = Math.min(100, bgIntensity + 30);
    hitSound.currentTime = 0;
    hitSound.play().catch(() => {});

    for (let i = 0; i < 15; i++) {
      const angle = Math.random() * Math.PI * 2;
      const speed = Math.random() * 12 + 8;
      particles.push({
        id: Date.now() + i,
        x: x + targetSize / 2,
        y: y + targetSize / 2,
        dx: Math.cos(angle) * speed,
        dy: Math.sin(angle) * speed,
        size: Math.random() * 8 + 6,
        life: 1,
        color: `hsl(${Math.random() * 20}, 100%, 60%)`
      });
    }

    randomPosition();
  }

  function miss() {
    lifes -= 1;
    bgIntensity = Math.max(0, bgIntensity - 20);
    missSound.currentTime = 0;
    missSound.play().catch(() => {});

    if (lifes <= 0) {
      clearInterval(interval);
      setTimeout(() => {
        alert(`GAME OVER! Final Score: ${score}`);
        reset();
      }, 300);
    } else {
      randomPosition();
    }
  }

  function reset() {
    score = 0;
    lifes = 5;
    randomPosition();
    startInterval();
  }

  function startInterval() {
    clearInterval(interval);
    interval = setInterval(() => {
      timeLeft -= 0.05;
      if (timeLeft <= 0) miss();
    }, 50);
  }

  function updateParticles() {
    particles = particles
      .map(p => {
        p.x += p.dx;
        p.y += p.dy;
        p.dy += 0.5;
        p.life -= 0.03;
        p.size *= 0.97;
        return p;
      })
      .filter(p => p.life > 0 && p.size > 1);

    animationFrame = requestAnimationFrame(updateParticles);
  }

  onMount(() => {
    randomPosition();
    startInterval();
    updateParticles();
  });

  onDestroy(() => {
    clearInterval(interval);
    cancelAnimationFrame(animationFrame);
  });

  $: bgColor = `linear-gradient(
    135deg,
    rgb(${20 + bgIntensity * 0.5}, ${20 + bgIntensity * 0.5}, ${40 + bgIntensity * 1.5}),
    rgb(${30 + bgIntensity * 0.5}, ${10 + bgIntensity * 0.5}, ${60 + bgIntensity * 1.2}),
    rgb(${40 + bgIntensity * 0.2}, ${0 + bgIntensity * 0.1}, ${80 + bgIntensity * 1.0})
  )`;
</script>

<svelte:window on:resize={() => particles = []} />

<main class="aim-trainer" style="background: {bgColor};" on:click={miss}>
  {#each particles as p (p.id)}
    <div
      class="explosion"
      style="
        left: {p.x}%;
        top: {p.y}%;
        width: {p.size}px;
        height: {p.size}px;
        background: {p.color};
        box-shadow: 0 0 {p.size * 2}px {p.color};
        opacity: {p.life};
      "
      out:fade={{ duration: 300 }}
    ></div>
  {/each}

  <div class="hud">
    <div class="stat score">
      <span class="label">SCORE</span>
      <span class="value">{score}</span>
    </div>
    <div class="stat lifes">
      <span class="label">LIFES</span>
      <span class="value">{lifes}</span>
    </div>
  </div>

  <div
    class="target"
    on:click={hit}
    style="top: {y}%; left: {x}%; width: {targetSize}px; height: {targetSize}px; transform: translate(-50%, -50%);"
  >
    <div class="core"></div>
    <div class="ring"></div>
    <div class="pulse"></div>
  </div>

  <div class="timebar-container">
    <div class="timebar" style="width: {timeLeft * 100}%"></div>
  </div>

  <button class="back-btn" on:click={() => dispatch('back')}>← BACK</button>
</main>

<style>
:global(body) { margin: 0; overflow: hidden; background: #111; }

.aim-trainer {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  font-family: 'Orbitron', sans-serif;
  color: white;
  cursor: crosshair;
  overflow: hidden;
  transition: background 0.4s ease;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
}

@keyframes gradientShift {
  0%{background-position:0% 50%;}
  50%{background-position:100% 50%;}
  100%{background-position:0% 50%;}
}

.hud {
  position: absolute;
  top: 5%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 2rem;
  z-index: 100;
}
.stat { background: rgba(0,0,0,0.5); padding: 0.8rem 1.8rem; border-radius: 15px; border: 2px solid; min-width: 120px; text-align: center; backdrop-filter: blur(10px); }
.score { border-color: #f39c12; }
.lifes { border-color: #e74c3c; }
.label { display: block; font-size: 0.8rem; letter-spacing: 0.1em; color: #aaa; text-transform: uppercase; }
.value { display: block; font-size: 2rem; font-weight: 900; text-shadow: 0 0 15px currentColor; }
.score .value { color: #f39c12; }
.lifes .value { color: #e74c3c; }

.target { position: absolute; border-radius: 50%; cursor: pointer; z-index: 10; filter: drop-shadow(0 0 20px #e74c3c); }
.core { position: absolute; top: 50%; left: 50%; width: 30%; height: 30%; background: #fff; border-radius: 50%; transform: translate(-50%, -50%); box-shadow: 0 0 30px #fff, inset 0 0 15px #e74c3c; }
.ring { position: absolute; top: 0; left: 0; right: 0; bottom: 0; border: 4px solid #e74c3c; border-radius: 50%; animation: rotate 2s linear infinite; }
.pulse { position: absolute; top: 50%; left: 50%; width: 100%; height: 100%; background: radial-gradient(circle, #e74c3c 0%, transparent 70%); border-radius: 50%; transform: translate(-50%, -50%); animation: pulse 1.5s infinite; opacity: 0.6; }
@keyframes rotate { to { transform: rotate(360deg); } }
@keyframes pulse { 0% { transform: translate(-50%, -50%) scale(0.8); opacity: 0.8; } 100% { transform: translate(-50%, -50%) scale(1.5); opacity: 0; } }

.timebar-container { position: absolute; bottom: 40px; width: 70%; max-width: 600px; height: 16px; background: rgba(0,0,0,0.5); border-radius: 8px; overflow: hidden; border: 1px solid rgba(255,255,255,0.2); z-index: 50; }
.timebar { height: 100%; background: linear-gradient(90deg, #f1c40f, #e74c3c); border-radius: 8px; transition: width 0.05s linear; }

.back-btn { position: absolute; top: 20px; left: 20px; padding: 0.8rem 1.6rem; background: rgba(0,0,0,0.6); color: white; border: 1px solid #e74c3c; border-radius: 10px; font-weight: bold; cursor: pointer; transition: all 0.3s ease; z-index: 100; }
.back-btn:hover { background: #e74c3c; transform: translateY(-3px); box-shadow: 0 0 20px #e74c3c; }

.explosion { position: absolute; border-radius: 50%; pointer-events: none; transform: translate(-50%, -50%); }

@media (max-width: 600px) {
  .hud { flex-direction: column; gap: 1rem; }
  .stat { padding: 0.6rem 1.2rem; min-width: 100px; }
  .value { font-size: 1.6rem; }
  .target { width: 60px; height: 60px; }
  .timebar-container { width: 85%; bottom: 20px; }
}
</style>

{#each Array(80) as _, i}
  <div class="particle" style="top:{Math.random()*100}%; left:{Math.random()*100}%; animation-duration:{5 + Math.random()*10}s; width:{1 + Math.random()*2}px; height:{1 + Math.random()*2}px;"></div>
{/each}
