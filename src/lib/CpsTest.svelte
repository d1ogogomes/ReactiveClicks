<script>
	import { createEventDispatcher } from "svelte";
	const dispatch = createEventDispatcher();

	let count = 0;
	let timeLeft = 10; // 10 segundos
	let timerRunning = false;
	let cps = 0;
	/**
	 * @type {string | number | NodeJS.Timeout | undefined}
	 */
	let interval;
	let intensity = 0; // barra visual tipo fogo

	function start() {
		if (timerRunning) return;

		count = 0;
		timeLeft = 10;
		cps = 0;
		intensity = 0;
		timerRunning = true;

		interval = setInterval(() => {
			timeLeft -= 0.1;
			if (timeLeft <= 0) {
				clearInterval(interval);
				timerRunning = false;
				// @ts-ignore
				cps = (count / 10).toFixed(2);
			}
			// diminui intensidade lentamente
			if (intensity > 0) intensity -= 0.5;
		}, 100);
	}

	function clickBtn() {
		if (!timerRunning) return;
		count += 1;
		intensity += 5; // aumenta barra de fogo a cada clique
		if (intensity > 100) intensity = 100; // limite máximo
	}
</script>

<main>
	<h1>CPS Test</h1>

	{#if timerRunning}
		<p>Time left: {timeLeft.toFixed(1)}s</p>
		<p>Clicks: {count}</p>
	{:else if cps > 0}
		<p>Your CPS: {cps}</p>
		<button on:click={start}>Try Again</button>
	{:else}
		<button on:click={start}>Start</button>
	{/if}

	<button
		on:click={clickBtn}
		disabled={!timerRunning}
		class="click-btn"
	>
		🔥 Click Me!
	</button>

	<div class="fire-bar">
		<div class="fire" style="width: {intensity}%"></div>
	</div>

	<button class="back-btn" on:click={() => dispatch("back")}>Back to Menu</button>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		gap: 1rem;
		font-family: 'Arial', sans-serif;
		text-align: center;
		background: #1e1e1e;
		color: white;
		user-select: none;
	}

	h1 {
		color: #f39c12;
		font-size: 2.5rem;
		text-shadow: 0 0 10px #f39c12;
	}

	.click-btn {
		width: 180px;
		height: 180px;
		border-radius: 50%;
		font-size: 1.5rem;
		background: linear-gradient(45deg, #e74c3c, #e67e22);
		color: white;
		border: none;
		cursor: pointer;
		box-shadow: 0 0 20px rgba(231,76,60,0.8);
		transition: transform 0.1s ease, box-shadow 0.1s ease;
	}
	.click-btn:active {
		transform: scale(0.9);
		box-shadow: 0 0 40px #f1c40f;
	}

	.fire-bar {
		width: 200px;
		height: 25px;
		background: rgba(255,255,255,0.1);
		border-radius: 10px;
		overflow: hidden;
		margin-top: 10px;
	}
	.fire {
		height: 100%;
		background: linear-gradient(90deg, #f1c40f, #e74c3c);
		transition: width 0.1s linear;
	}

	button {
		margin-top: 1rem;
		padding: 0.6rem 1.2rem;
		border-radius: 8px;
		cursor: pointer;
		background: #3498db;
		color: rgb(248, 248, 248);
		border: none;
		transition: 0.2s;
	}
	button:hover {
		transform: scale(1.05);
		background: #2980b9;
	}
</style>
