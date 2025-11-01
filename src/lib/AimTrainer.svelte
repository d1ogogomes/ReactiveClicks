<script>
	import { createEventDispatcher, onMount, onDestroy } from "svelte";
	const dispatch = createEventDispatcher();

	let score = 0;
	let lifes = 5;
	let x = 50;
	let y = 50;
	let timeLeft = 1;
	let interval;
	const targetSize = 50;

	function randomPosition() {
		x = Math.random() * 80 + 10;
		y = Math.random() * 60 + 20;
		timeLeft = 1;
	}

	function hit(event) {
		event.stopPropagation();
		score += 1;
		randomPosition();
	}

	function startInterval() {
    clearInterval(interval);
    interval = setInterval(() => {
        timeLeft -= 0.05;
        if (timeLeft <= 0) miss();
    }, 50);
}

function miss() {
    lifes -= 1;
    if (lifes <= 0) {
        clearInterval(interval);
        alert(`Game Over! Your score: ${score}`);
        score = 0;
        lifes = 5;
        randomPosition();
        startInterval(); // reinicia o timer
    } else {
        randomPosition();
    }
}

onMount(() => {
    startInterval();
});

onDestroy(() => {
    clearInterval(interval);
});

</script>

<main on:click={miss}>
	<div class="hud">
		<p>Score: {score}</p>
		<p>Lifes: {lifes}</p>
	</div>

	<div
		class="target"
		on:click={hit}
		style="top: {y}%; left: {x}%; width: {targetSize}px; height: {targetSize}px;">
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
		background: linear-gradient(135deg, #1f1c2c, #928dab);
		display: flex;
		align-items: center;
		justify-content: center;
		font-family: 'Arial', sans-serif;
		color: white;
		user-select: none;
		overflow: hidden;
	}

	.hud {
		position: absolute;
		top: 5%;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		gap: 2rem;
		font-size: 1.5rem;
		background: rgba(0,0,0,0.2);
		padding: 0.5rem 1rem;
		border-radius: 12px;
		backdrop-filter: blur(5px);
	}

	.target {
		position: absolute;
		border-radius: 50%;
		cursor: pointer;
		box-shadow: 0 0 20px #e74c3c, 0 0 40px rgba(231,76,60,0.5) inset;
		transition: transform 0.1s ease, box-shadow 0.2s ease;
		background: radial-gradient(circle, #e74c3c 0%, #c0392b 70%);
	}
	.target:active {
		transform: scale(0.8);
		box-shadow: 0 0 30px #f1c40f, 0 0 50px rgba(241,196,15,0.6) inset;
	}

	.timebar {
		position: absolute;
		bottom: 30px;
		width: 80%;
		height: 20px;
		background: rgba(255,255,255,0.2);
		border-radius: 10px;
		overflow: hidden;
		box-shadow: inset 0 0 5px rgba(0,0,0,0.3);
	}
	.time {
		height: 100%;
		background: linear-gradient(90deg, #f1c40f, #e74c3c);
		transition: width 0.05s linear;
	}

	.back-btn {
		position: absolute;
		top: 10px;
		left: 10px;
		padding: 0.6rem 1.2rem;
		border-radius: 8px;
		border: none;
		background: #e67e22;
		color: white;
		font-weight: bold;
		cursor: pointer;
		transition: transform 0.2s ease, box-shadow 0.2s ease;
	}
	.back-btn:hover {
		transform: scale(1.05);
		box-shadow: 0 0 10px #f1c40f;
	}
</style>
