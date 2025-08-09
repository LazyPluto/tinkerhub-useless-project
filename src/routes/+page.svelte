<script lang="ts">
	import Keyboard from '$lib/components/Keyboard.svelte';
	import { onMount } from 'svelte';

	const TestState = {
		NOT_STARTED: 'NOT_STARTED',
		RUNNING: 'RUNNING',
		ENDED: 'ENDED'
	};

	let words = prepareWords(
		'The quick brown fox jumps over the lazy dog The quick brown fox jumps over the lazy dog The quick brown fox jumps over the lazy dog'
	);
	let letterStates = $state(prepareStates(words));
	// let keymap = $state(prepareDefaultKeymap(0.5));
	let keymap = $state(prepareABCDKeymap());
	let timeLeft = $state(0);
	let nextWord = 0;
	let nextChar = 0;
	let lettersTyped = 0;
	let startTime: Date;
	let endTime: Date;
	let testState = $state(TestState.NOT_STARTED);
	let wpm = $state(0);
	let timer: number;

	function prepareWords(sentence: string): string[][] {
		const words = sentence.toLowerCase().split(' ');
		const result = words.map((word) => word.split(''));
		return result;
	}

	function prepareStates(words: string[][]): boolean[][] {
		return words.map((word) => word.map((_) => false));
	}

	function prepareDefaultKeymap(difficulty: number): Map<string, string> {
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
		testState = TestState.NOT_STARTED;
		wpm = 0;
	}

	onMount(() => {
		window.addEventListener('keydown', (event) => {
			if (testState === TestState.NOT_STARTED) startTest();
			if (testState === TestState.RUNNING) handleTyping(event);
		});
	});
</script>

<div class="test-container">
	<h1>Typing Speed Test</h1>

	<div class="stats {testState === TestState.NOT_STARTED ? 'hidden' : ''}">
		<p>{timeLeft}s</p>
		<p>WPM: {wpm}</p>
	</div>

	<div class="words">
		{#each words as word, wordIndex}
			<div class="word">
				{#each word as char, charIndex}
					<div class="char {letterStates[wordIndex][charIndex] ? 'typed' : ''}">{char}</div>
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

	.stats {
		width: 75%;
		margin-top: 8%;
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
		height: 40%;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		align-content: flex-start;
		overflow: hidden;
		gap: 0.5rem 1rem;
		font-size: 2.5rem;
		margin-bottom: 20px;
		background: transparent;
		border-radius: 4px;
		color: #45475a;
	}

	.word {
		display: flex;
		flex-direction: row;
	}

	.char.typed {
		color: #cdd6f4;
	}
</style>
