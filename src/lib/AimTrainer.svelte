<script>
	import { createEventDispatcher, onMount, onDestroy } from "svelte";
	const dispatch = createEventDispatcher();

	let score = 0;
	let lifes = 5;
	let x = 50;
	let y = 50;
	let timeLeft = 1; // tempo restante do alvo em segundos
	/**
	 * @type {string | number | NodeJS.Timeout | undefined}
	 */
	let interval;

	const targetSize = 50; // tamanho do alvo

	function randomPosition() {
		x = Math.random() * (100 - 20) + 10;
		y = Math.random() * (100 - 20) + 10;
		timeLeft = 1; // reinicia o tempo
	}

	/**
	 * @param {{ stopPropagation: () => void; }} event
	 */
	function hit(event) {
		event.stopPropagation();
		score += 1;
		randomPosition();
	}

	// @ts-ignore
	function miss() {
		lifes -= 1;
		if (lifes <= 0) {
			clearInterval(interval);
			alert(`Game Over! Your score: ${score}`);
			score = 0;
			lifes = 5;
			randomPosition();
		} else {
			randomPosition();
		}
	}

	onMount(() => {
		// countdown do alvo
		interval = setInterval(() => {
			timeLeft -= 0.05;
			if (timeLeft <= 0) {
				miss();
			}
		}, 50);
	});

	onDestroy(() => {
		clearInterval(interval);
	});
</script>

<main>
	<div class="hud">
		<p>Score: {score}</p>
		<p>Lifes: {lifes}</p>
	</div>

	<!-- svelte-ignore a11y_click_events_have_key_events -->
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div
		class="target"
		on:click={hit}
		style="top: {y}%; left: {x}%; width: {targetSize}px; height: {targetSize}px; background: radial-gradient(circle, #e74c3c 0%, #c0392b 70%)">
	</div>

	<div class="timebar">
		<div class="time" style="width: {timeLeft * 100}%"></div>
	</div>

	<button class="back-btn" on:click={() => dispatch("back")}>Back to Menu</button>
</main>

<style>
	main {
		position: relative;
		width: 100%;
		height: 100vh;
		background: #2c3e50;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		font-family: 'Arial', sans-serif;
		user-select: none;
		color: white;
		overflow: hidden;
	}

	.hud {
		position: absolute;
		top: 10px;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		gap: 2rem;
		font-size: 1.2rem;
	}

	.target {
		position: absolute;
		border-radius: 50%;
		cursor: pointer;
		box-shadow: 0 0 15px rgba(0,0,0,0.5);
		transition: transform 0.1s ease;
	}
	.target:active {
		transform: scale(0.8);
	}

	.timebar {
		position: absolute;
		bottom: 20px;
		width: 80%;
		height: 20px;
		background: rgba(255,255,255,0.2);
		border-radius: 10px;
		overflow: hidden;
	}
	.time {
		height: 100%;
		background: #f1c40f;
		transition: width 0.05s linear;
	}

	.back-btn {
		position: absolute;
		top: 10px;
		left: 10px;
		padding: 0.5rem 1rem;
		border-radius: 6px;
		border: none;
		background: #3498db;
		color: white;
		cursor: pointer;
		font-weight: bold;
		transition: 0.2s;
	}
	.back-btn:hover {
		transform: scale(1.05);
		background: #2980b9;
	}
</style>
