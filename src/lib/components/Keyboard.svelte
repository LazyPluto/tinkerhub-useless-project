<script>
	import { onMount } from 'svelte';

	let pressedKey = null;
	let keys = [
		['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
		['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
		['Z', 'X', 'C', 'V', 'B', 'N', 'M']
	];

	// Function to handle the key press event
	function handleKeyPress(event) {
		pressedKey = event.key.toUpperCase();
	}

	// Listen to keydown events
	onMount(() => {
		window.addEventListener('keydown', handleKeyPress);
		return () => {
			window.removeEventListener('keydown', handleKeyPress);
		};
	});
</script>

<div class="keyboard">
	{#each keys as row}
		<div class="keyrow">
			{#each row as key}
				<div class="key">
					{key}
				</div>
			{/each}
		</div>
	{/each}
</div>

<style>
	.keyboard {
		display: flex;
		flex-direction: column;
		max-width: 900px;
		margin: auto;
		text-align: center;
		align-items: center;
	}

	.keyrow {
		display: flex;
		flex-direction: row;
		align-items: center;
	}

	.key {
		aspect-ratio: 1 / 1;
		padding: 10px;
		margin: 5px;
		border: 2px solid #ccc;
		border-radius: 4px;
		background-color: #f0f0f0;
		cursor: pointer;
		transition: background-color 0.3s;
		font-size: 14px;
		font-family: 'Courier New', monospace;
	}

	.key.pressed {
		background-color: #4caf50;
		color: white;
	}
</style>
