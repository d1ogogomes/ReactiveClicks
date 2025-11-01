<script>
	import CpsTest from '$lib/CpsTest.svelte';
	import AimTrainer from '$lib/AimTrainer.svelte';

	let page = "menu"; // "menu", "cps", "aim"

	/**
	 * @param {string} target
	 */
	function goTo(target) {
		page = target;
	}
	
</script>

{#if page === "menu"}
	<main class="menu">
		<h1>Reactive Clicks</h1>
		<div class="buttons">
			<button on:click={() => goTo("cps")}>CPS Test</button>
			<button on:click={() => goTo("aim")}>Aim Trainer</button>
		</div>
	</main>
{/if}

{#if page === "cps"}
	<CpsTest on:back={() => goTo("menu")} />
{/if}

{#if page === "aim"}
	<AimTrainer on:back={() => goTo("menu")} />
{/if}

<style>
	main.menu {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		gap: 2rem;
		font-family: 'Arial', sans-serif;
		text-align: center;
		background: linear-gradient(135deg, #1f1c2c, #928dab);
		color: white;
	}

	h1 {
		font-size: 3rem;
		text-transform: uppercase;
		letter-spacing: 2px;
		color: #f39c12;
		text-shadow: 0 0 10px #f39c12, 0 0 20px #f39c12;
	}

	.buttons {
		display: flex;
		flex-direction: column;
		gap: 1.5rem;
	}

	button {
		padding: 1rem 3rem;
		font-size: 1.5rem;
		cursor: pointer;
		border-radius: 12px;
		background: linear-gradient(45deg, #e74c3c, #e67e22);
		color: white;
		border: none;
		box-shadow: 0 5px 15px rgba(231, 76, 60, 0.6);
		transition: transform 0.2s ease, box-shadow 0.2s ease, background 0.3s ease;
	}

	button:hover {
		transform: scale(1.1);
		box-shadow: 0 8px 20px rgba(231, 76, 60, 0.8);
		background: linear-gradient(45deg, #f1c40f, #e74c3c);
	}

	button:active {
		transform: scale(0.95);
		box-shadow: 0 3px 10px rgba(231, 76, 60, 0.6);
	}
</style>
