<script>
  import CpsTest from '$lib/CpsTest.svelte';
  import AimTrainer from '$lib/AimTrainer.svelte';
  import ReactionTest from '$lib/ReactionTest.svelte';
  import { onMount, onDestroy } from 'svelte';
  import { fade, fly } from 'svelte/transition';
  import { quintOut } from 'svelte/easing';

  let page = 'menu';
  let particles = [];

  function goTo(target) {
    page = target;
  }

  let animationFrame;

  function updateParticles() {
    particles = particles
      .map(p => ({
        ...p,
        x: p.x + p.dx,
        y: p.y + p.dy
      }))
      .filter(p => p.x >= -50 && p.x <= window.innerWidth + 50 && p.y >= -50 && p.y <= window.innerHeight + 50);

    animationFrame = requestAnimationFrame(updateParticles);
  }

  // Gerar partículas flutuantes
  onMount(() => {
    const interval = setInterval(() => {
      const newParticle = {
        id: Date.now() + Math.random(),
        x: Math.random() * window.innerWidth,
        y: Math.random() * window.innerHeight,
        size: Math.random() * 6 + 4,
        dx: (Math.random() - 0.5) * 2, // horizontal random speed
        dy: (Math.random() - 0.5) * 2  // vertical random speed
      };
      particles = [...particles, newParticle].slice(-50); // máximo 50 partículas
    }, 200);

    updateParticles();

    return () => {
      clearInterval(interval);
      cancelAnimationFrame(animationFrame);
    };
  });
</script>

<svelte:window on:resize={() => particles = []} />

{#if page === 'menu'}
  <main class="menu" in:fly={{ y: 50, duration: 800, easing: quintOut }}>
    {#each particles as p (p.id)}
      <div
        class="particle"
        style="
          left: {p.x}px;
          top: {p.y}px;
          width: {p.size}px;
          height: {p.size}px;
        "
        out:fade={{ duration: 500 }}
      ></div>
    {/each}

    <div class="logo">
      <h1>REACTIVE<span>CLICKS</span></h1>
      <div class="glow"></div>
    </div>

    <div class="buttons">
      <button on:click={() => goTo('cps')} class="neon-btn red"><span>CPS TEST</span><div class="spark"></div></button>
      <button on:click={() => goTo('aim')} class="neon-btn yellow"><span>AIM TRAINER</span><div class="spark"></div></button>
      <button on:click={() => goTo('reaction')} class="neon-btn green"><span>F1 REACTION</span><div class="spark"></div></button>
    </div>

    <p class="tagline">Train like a pro. Click like a god.</p>
  </main>
{/if}

{#if page === 'cps'}
  <CpsTest on:back={() => goTo('menu')} />
{/if}
{#if page === 'aim'}
  <AimTrainer on:back={() => goTo('menu')} />
{/if}
{#if page === 'reaction'}
  <ReactionTest on:back={() => goTo('menu')} />
{/if}

<style>
  :global(body) {
    margin: 0;
    overflow: hidden;
    background: #0a0a0a;
    font-family: 'Orbitron', 'Arial', sans-serif;
  }

  .menu {
    position: relative;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: radial-gradient(circle at center, #1a1a2e, #0f0f1e);
    overflow: hidden;
    color: white;
  }

  .particle {
    position: absolute;
    background: #f39c12;
    border-radius: 50%;
    box-shadow: 0 0 10px #f39c12, 0 0 20px #f39c12;
    pointer-events: none;
    opacity: 0.7;
  }

  .logo {
    position: relative;
    margin-bottom: 3rem;
    text-align: center;
  }

  .logo h1 {
    font-size: 4.5rem;
    font-weight: 900;
    letter-spacing: 0.2em;
    background: linear-gradient(45deg, #f39c12, #e74c3c, #f1c40f);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 30px rgba(241, 196, 15, 0.5);
  }

  .logo span {
    display: block;
    font-size: 1.8rem;
    letter-spacing: 0.4em;
    color: #aaa;
    margin-top: -0.5rem;
  }

  .glow {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 300px;
    height: 300px;
    background: radial-gradient(circle, #f39c12 0%, transparent 70%);
    filter: blur(40px);
    transform: translate(-50%, -50%);
    opacity: 0.3;
    animation: pulse 3s infinite;
  }

  @keyframes pulse {
    0%, 100% { transform: translate(-50%, -50%) scale(1); }
    50% { transform: translate(-50%, -50%) scale(1.2); }
  }

  .buttons {
    display: flex;
    flex-direction: column;
    gap: 1.8rem;
    z-index: 10;
  }

  .neon-btn {
    position: relative;
    padding: 1.2rem 3.5rem;
    font-size: 1.6rem;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    border: none;
    border-radius: 15px;
    cursor: pointer;
    overflow: hidden;
    transition: all 0.3s ease;
    transform-style: preserve-3d;
  }

  .neon-btn span {
    position: relative;
    z-index: 2;
    background: linear-gradient(45deg, #fff, #ddd);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .neon-btn.red::before { background: linear-gradient(45deg, #e74c3c, #c0392b); }
  .neon-btn.yellow::before { background: linear-gradient(45deg, #f39c12, #e67e22); }
  .neon-btn.green::before { background: linear-gradient(45deg, #2ecc71, #27ae60); }

  .tagline {
    margin-top: 3rem;
    font-size: 1.1rem;
    color: #666;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }

  @media (max-width: 600px) {
    .logo h1 { font-size: 3rem; }
    .neon-btn { padding: 1rem 2.5rem; font-size: 1.3rem; }
  }
</style>
