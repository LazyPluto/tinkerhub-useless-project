<script lang="ts">
	import { onMount } from 'svelte';

	let { keymap } = $props();

	let keys = [
		['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
		['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L'],
		['Z', 'X', 'C', 'V', 'B', 'N', 'M', ',']
	];

	let keyStates = $state(createKeyStates(keys));

	function createKeyStates(keys: string[][]): Map<string, boolean> {
		const map = new Map<string, boolean>();
		keys.forEach((innerArray) => {
			innerArray.forEach((char) => {
				map.set(char, false);
			});
		});
		return map;
	}

	function handleKeyPress(event: KeyboardEvent) {
		keyStates.set(event.key.toUpperCase(), true);
		keyStates = new Map(keyStates);
	}

	function handleKeyRelease(event: KeyboardEvent) {
		keyStates.set(event.key.toUpperCase(), false);
		keyStates = new Map(keyStates);
	}

	onMount(() => {
		window.addEventListener('keydown', handleKeyPress);
		window.addEventListener('keyup', handleKeyRelease);
		return () => {
			window.removeEventListener('keydown', handleKeyPress);
		};
	});
</script>

<div class="keyboard">
	{#each keys as row}
		<div class="keyrow">
			{#each row as key}
				<div class="key {keyStates.get(key) ? 'pressed' : ''}">
					{keymap.get(key)}
				</div>
			{/each}
		</div>
	{/each}
</div>

<style>
	.keyboard {
		display: flex;
		flex-direction: column;
		gap: 0.3em;
		max-width: 900px;
		text-align: center;
		align-items: center;
	}

	.keyrow {
		display: flex;
		flex-direction: row;
		align-items: center;
		gap: 0.4em;
	}

	.key {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 3em;
		height: 3em;
		border: 2px solid #6c7086;
		border-radius: 4px;
		background-color: #313244;
		cursor: pointer;
		transition: background-color 0.15s;
		font-size: 16pt;
		font-family: 'Roboto Mono', monospace;
	}

	.key.pressed {
		background-color: #b4befe;
		color: white;
	}
</style>
