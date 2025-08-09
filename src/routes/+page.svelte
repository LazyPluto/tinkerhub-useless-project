<script lang="ts">
	import Keyboard from '$lib/components/Keyboard.svelte';
	import { onMount } from 'svelte';
	import { WORDS } from '$lib/wordlists/english';

	const TestState = {
		NOT_STARTED: 'NOT_STARTED',
		RUNNING: 'RUNNING',
		ENDED: 'ENDED'
	};

	const LayoutType = {
		ALPHABETIC: 'ALPHABETIC',
		RANDOM: 'RANDOM'
	};

	// let words = prepareWords(
	// 	'The quick brown fox jumps over the lazy dog The quick brown fox jumps over the lazy dog The quick brown fox jumps over the lazy dog'
	// );
	let words = $state(prepareWords());
	let letterStates = $derived(prepareStates(words));
	let keymapType = $state(LayoutType.ALPHABETIC);
	// let keymap = $state(prepareRandomKeymap(0.5));
	// let keymap = $state(prepareRandomKeymap(0));
	let keymap = $derived(
		keymapType === LayoutType.ALPHABETIC ? prepareABCDKeymap() : prepareRandomKeymap(0.9)
	);
	let timeLeft = $state(0);
	let nextWord = $state(0);
	let nextChar = $state(0);
	let lettersTyped = 0;
	let startTime: Date;
	let endTime: Date;
	let testState = $state(TestState.NOT_STARTED);
	let wpm = $state(0);
	let timer: number;

	function prepareWords(): string[][] {
		const words = [];
		for (let i = 0; i < 200; i++) {
			const word = WORDS[Math.floor(Math.random() * WORDS.length)].toLowerCase().split('');
			words.push(word);
		}

		return words;
	}

	function prepareStates(words: string[][]): boolean[][] {
		return words.map((word) => word.map((_) => false));
	}

	function prepareRandomKeymap(difficulty: number): Map<string, string> {
		const letters = [];
		for (let i = 65; i <= 90; i++) {
			letters.push(i);
		}

		// for (let i = letters.length - 1; i > 0; i--) {
		// 	const j = Math.floor(Math.random() * (i + 1));
		// 	[letters[i], letters[j]] = [letters[j], letters[i]];
		// }

		const rounds = Math.round(letters.length * difficulty);
		for (let round = 0; round < rounds; round++) {
			const i = Math.floor(Math.random() * letters.length);
			const j = Math.floor(Math.random() * (i + 1));
			[letters[i], letters[j]] = [letters[j], letters[i]];
		}

		const letterMap = new Map<string, string>();

		for (let i = 0; i < letters.length; i++) {
			letterMap.set(String.fromCharCode(65 + i), String.fromCharCode(letters[i]));
		}

		return letterMap;
	}

	function prepareABCDKeymap(): Map<string, string> {
		const letters = [
			'Q',
			'W',
			'E',
			'R',
			'T',
			'Y',
			'U',
			'I',
			'O',
			'P',
			'A',
			'S',
			'D',
			'F',
			'G',
			'H',
			'J',
			'K',
			'L',
			'Z',
			'X',
			'C',
			'V',
			'B',
			'N',
			'M'
		];

		const letterMap = new Map<string, string>();
		for (let i = 0; i < letters.length; i++) {
			letterMap.set(letters[i], String.fromCharCode(65 + i));
		}
		return letterMap;
	}

	function startTest() {
		console.log('Starting test');
		testState = TestState.RUNNING;
		startTime = new Date();
		endTime = new Date(startTime.getTime() + 30 * 1000);
		timer = setInterval(updateWPM, 1000);
	}

	function stopTest() {
		console.log('Stopping test');
		testState = TestState.ENDED;
		clearInterval(timer);
		timeLeft = 0;
	}

	function updateWPM() {
		const time = new Date();
		const timeTaken = (time - startTime) / 1000;
		const wordsTyped = lettersTyped / 5;
		wpm = Math.round((wordsTyped / timeTaken) * 60);
		timeLeft = Math.floor(120 - timeTaken);

		if (timeTaken >= 120) stopTest();
	}

	// Update the typed text and check if the test is finished
	function handleTyping(event: KeyboardEvent) {
		let key = event.key.toUpperCase();
		key = keymap.get(key)?.toLowerCase();

		if (event.key == ' ') {
			if (nextChar != 0) {
				nextWord++;
				nextChar = 0;
			}
		} else {
			if (nextChar < words[nextWord].length) {
				if (words[nextWord][nextChar] === key) {
					letterStates[nextWord][nextChar] = true;
					lettersTyped++;
					nextChar++;
				}
			}
		}
	}

	// Reset the test
	function resetTest() {
		if (testState === TestState.RUNNING) stopTest();

		words = prepareWords();
		testState = TestState.NOT_STARTED;
		wpm = 0;
	}

	function switchLayoutState() {
		if (keymapType === LayoutType.ALPHABETIC) keymapType = LayoutType.RANDOM;
		else keymapType = LayoutType.ALPHABETIC;
	}

	onMount(() => {
		window.addEventListener('keydown', (event) => {
			if (testState === TestState.NOT_STARTED) startTest();
			if (testState === TestState.RUNNING) handleTyping(event);
		});
	});
</script>

<svelte:head>
	<title>Chaos Keys</title>
</svelte:head>

<div class="test-container">
	<h1>Chaos Keys</h1>

	<div class="controls">
		<!-- <button onclick={() => switchLayoutState()} -->
		<!-- 	>{keymapType === LayoutType.ALPHABETIC ? 'Alphabetic' : 'Random'}</button -->
		<!-- > -->
		<!-- <button onclick={() => resetTest()}>Reset</button> -->
		<button onclick={() => switchLayoutState()}>
			<svg
				width="24"
				height="24"
				viewBox="0 0 576 512"
				fill="currentColor"
				stroke="currentColor"
				stroke-width="2"
				xmlns="http://www.w3.org/2000/svg"
				><path
					d="M512 64H64C28.65 64 0 92.65 0 128v256c0 35.35 28.65 64 64 64h448c35.35 0 64-28.65 64-64V128C576 92.65 547.3 64 512 64zM528 384c0 8.822-7.178 16-16 16H64c-8.822 0-16-7.178-16-16V128c0-8.822 7.178-16 16-16h448c8.822 0 16 7.178 16 16V384zM140 152h-24c-6.656 0-12 5.344-12 12v24c0 6.656 5.344 12 12 12h24c6.656 0 12-5.344 12-12v-24C152 157.3 146.7 152 140 152zM196 200h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C184 194.7 189.3 200 196 200zM276 200h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C264 194.7 269.3 200 276 200zM356 200h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C344 194.7 349.3 200 356 200zM460 152h-24c-6.656 0-12 5.344-12 12v24c0 6.656 5.344 12 12 12h24c6.656 0 12-5.344 12-12v-24C472 157.3 466.7 152 460 152zM140 232h-24c-6.656 0-12 5.344-12 12v24c0 6.656 5.344 12 12 12h24c6.656 0 12-5.344 12-12v-24C152 237.3 146.7 232 140 232zM196 280h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C184 274.7 189.3 280 196 280zM276 280h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C264 274.7 269.3 280 276 280zM356 280h24c6.656 0 12-5.344 12-12v-24c0-6.656-5.344-12-12-12h-24c-6.656 0-12 5.344-12 12v24C344 274.7 349.3 280 356 280zM460 232h-24c-6.656 0-12 5.344-12 12v24c0 6.656 5.344 12 12 12h24c6.656 0 12-5.344 12-12v-24C472 237.3 466.7 232 460 232zM400 320h-224C167.1 320 160 327.1 160 336V352c0 8.875 7.125 16 16 16h224c8.875 0 16-7.125 16-16v-16C416 327.1 408.9 320 400 320z"
				/></svg
			>
		</button>
		<button onclick={() => resetTest()}>
			<svg
				width="24"
				height="24"
				viewBox="0 0 200 200"
				fill="currentColor"
				stroke="currentColor"
				stroke-width="2"
				xmlns="http://www.w3.org/2000/svg"
				><path
					d="M182.5,40a10,10,0,0,0-10-10h-30a20.06,20.06,0,0,0-20,20V80a10,10,0,0,0,20,0V60.5a58.74,58.74,0,0,1,15,39.5,60,60,0,1,1-60-60,10,10,0,0,0,0-20,80,80,0,1,0,80,80A78.93,78.93,0,0,0,160,50h12.5A10,10,0,0,0,182.5,40Z"
				/></svg
			>
		</button>
	</div>

	<div class="stats {testState === TestState.NOT_STARTED ? 'hidden' : ''}">
		<p>{timeLeft}s</p>
		<p>WPM: {wpm}</p>
	</div>

	<div class="words">
		{#each words as word, wordIndex}
			<div class="word">
				{#each word as char, charIndex}
					<div
						class="char
							{letterStates[wordIndex][charIndex] ? 'typed' : ''}
							{nextWord === wordIndex && nextChar === charIndex ? 'active' : ''}
							{charIndex === word.length - 1 && nextWord === wordIndex && nextChar >= word.length
							? 'last-active'
							: ''}"
					>
						{char}
					</div>
				{/each}
			</div>
		{/each}
	</div>
	<Keyboard {keymap} />
</div>

<style>
	.test-container {
		width: 100vw;
		height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		margin: auto;
		text-align: center;
		padding: 20px;
		font-family: Arial, sans-serif;
		background-color: #1e1e2e;
		color: #cdd6f4;
	}

	h1 {
		font-size: 32pt;
		margin: 0;
	}

	.controls {
		display: flex;
		gap: 8px;
		padding: 2px 4px;
	}

	.controls button {
		padding: 8px;
		color: #45475a;
	}

	.controls button:hover {
		color: #cdd6f4;
	}

	.stats {
		width: 75%;
		margin-top: 2vh;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		font-size: 16pt;
	}

	.hidden {
		opacity: 0;
	}

	.words {
		width: 75%;
		height: 35%;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		align-content: flex-start;
		overflow: hidden;
		gap: 0rem 1rem;
		font-size: 2.5rem;
		margin-bottom: 20px;
		background: transparent;
		border-radius: 4px;
		color: #45475a;
		font-family: monospace;
	}

	.word {
		display: flex;
		flex-direction: row;
	}

	.char {
		display: flex;
		flex-direction: row;
	}

	.char.typed {
		color: #cdd6f4;
	}

	.char.active::before,
	.char.last-active::after {
		content: '';
		display: inline;
		width: 2px;
		height: 65%;
		margin-top: 0.15em;
		display: block;
		background-color: #f9e2af;
	}
</style>
