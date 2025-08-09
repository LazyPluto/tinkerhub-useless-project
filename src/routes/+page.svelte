<script>
	import Keyboard from '$lib/components/Keyboard.svelte';

	let sentence = 'The quick brown fox jumps over the lazy dog';
	let typedText = '';
	let startTime;
	let endTime;
	let isTestStarted = false;
	let isTestFinished = false;
	let timeTaken = 0;
	let wpm = 0;

	// Start a new test
	function startTest() {
		typedText = '';
		isTestStarted = true;
		isTestFinished = false;
		startTime = new Date();
	}

	// Update the typed text and check if the test is finished
	function handleTyping(event) {
		typedText = event.target.value;

		if (typedText === sentence) {
			endTime = new Date();
			isTestFinished = true;
			timeTaken = (endTime - startTime) / 1000; // in seconds
			wpm = Math.round((sentence.split(' ').length / timeTaken) * 60);
		}
	}

	// Reset the test
	function resetTest() {
		typedText = '';
		isTestStarted = false;
		isTestFinished = false;
		timeTaken = 0;
		wpm = 0;
	}
</script>

<div class="test-container">
	<h1>Typing Speed Test</h1>

	{#if !isTestStarted}
		<p>Click below to start the test.</p>
		<button on:click={startTest}>Start Test</button>
	{/if}

	{#if isTestStarted}
		<p class="sentence">{sentence}</p>
		<input type="text" bind:value={typedText} on:input={handleTyping} disabled={isTestFinished} />
	{/if}

	{#if isTestFinished}
		<div class="result">
			<p>Time Taken: {timeTaken} seconds</p>
			<p>Words per Minute: {wpm} WPM</p>
			<button on:click={resetTest}>Try Again</button>
		</div>
	{/if}

	<Keyboard />
</div>

<style>
	.test-container {
		width: 100%;
		max-width: 600px;
		margin: auto;
		text-align: center;
		padding: 20px;
		font-family: Arial, sans-serif;
	}

	.sentence {
		font-size: 1.5rem;
		margin-bottom: 20px;
		padding: 10px;
		background: #f3f3f3;
		border-radius: 4px;
	}

	input {
		width: 100%;
		padding: 10px;
		font-size: 1.2rem;
		margin-top: 10px;
		border: 2px solid #ccc;
		border-radius: 4px;
	}

	.result {
		margin-top: 20px;
	}

	button {
		margin-top: 20px;
		padding: 10px 20px;
		font-size: 1rem;
		cursor: pointer;
		border: none;
		background-color: #4caf50;
		color: white;
		border-radius: 4px;
	}

	button:hover {
		background-color: #45a049;
	}
</style>
