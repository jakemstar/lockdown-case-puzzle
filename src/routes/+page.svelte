<script lang="ts">
	import { onMount } from 'svelte';

	const maxAttempts = 9;
	const statusToColor: Record<string, string> = {
		Y: 'sky-500',
		N: 'red-400',
		M: 'lime-300'
	};

	let secretNumber = $state(generateRandomFourDigitString());
	let priorAttempt = $state(['', '', '', '']);
	let priorAttemptCorrect = $state(['', '', '', '']);
	let currentAttempt = $state(['', '', '', '']);
	let attempts = $state(0);

	let inputFilled = $derived(currentAttempt.every((val) => val !== ''));
	let remainingAttempts = $derived(maxAttempts - attempts);
	let gameWon = $derived(priorAttemptCorrect.every((status) => status === 'Y'));
	let gameOver = $derived(remainingAttempts <= 0 || gameWon);
	let priorAttemptColors = $derived(
		priorAttemptCorrect.map((status) => statusToColor[status] || 'gray')
	);

	$effect(() => {
		if (inputFilled) {
			const submitButton = document.getElementById('submit-button');
			submitButton?.focus();
		}
	});

	$effect(() => {
		if (gameOver) {
			const restartButton = document.getElementById('restart-button');
			restartButton?.focus();
		}
	});

	onMount(() => {
		const firstInput = document.getElementById('input-0');
		if (firstInput) {
			firstInput.focus();
		}
	});

	function handleInput(index: number, value: string) {
		if (gameOver) return;

		if (value) {
			currentAttempt[index] = value;
		}
		if (value && index < 3) {
			const nextInput = document.getElementById(`input-${index + 1}`);
			if (nextInput) {
				nextInput.focus();
			}
		}
	}

	function handleNumberPad(number: number | undefined) {
		if (number === undefined) return;
		const emptyIndex = currentAttempt.findIndex((val) => val === '');
		if (emptyIndex !== -1) {
			handleInput(emptyIndex, number.toString());
		}
	}

	function handleSubmit() {
		if (gameOver) return;

		const firstInput = document.getElementById('input-0');
		if (!firstInput) return;

		if (currentAttempt.every((val) => val !== '')) {
			priorAttempt = [...currentAttempt];
			currentAttempt = ['', '', '', ''];
			attempts++;
			firstInput.focus();
		}

		priorAttemptCorrect = evaluateGuess(priorAttempt.join(''));
	}

	function generateRandomFourDigitString() {
		let result = '';
		const usedDigits = new Set();
		while (result.length < 4) {
			const randomDigit = Math.floor(Math.random() * 10); // Generates a random number between 0 and 9
			if (!usedDigits.has(randomDigit)) {
				usedDigits.add(randomDigit);
				result += randomDigit.toString();
			}
		}
		return result;
	}

	function evaluateGuess(guess: string) {
		const guessArray = guess.split('');
		const answerArray = secretNumber.toString().split('');

		const result = Array(4).fill('N');

		// Set all digits in correct positions to 'Y'
		guessArray.forEach((digit, i) => {
			if (digit === answerArray[i]) {
				result[i] = 'Y';
				// Prevent this value from being checked again
				answerArray[i] = 'X';
			}
		});

		// Count number of times each digit appears in the wrong position
		const guessArrayNoDupes = [...new Set(guessArray)];
		const digitCountInWrongPosition: Record<string, number> = {};
		guessArrayNoDupes.forEach((digit) => {
			digitCountInWrongPosition[digit] = answerArray.filter(
				(answerDigit) => answerDigit === digit
			).length;
		});

		// Set all digits in the wrong positions to 'M'
		for (let i = 0; i < guessArray.length; i++) {
			if (result[i] === 'N' && digitCountInWrongPosition[guessArray[i]] > 0) {
				for (let j = 0; j < 4; j++) {
					if (guessArray[i] === answerArray[j]) {
						result[i] = 'M';
						digitCountInWrongPosition[guessArray[i]]--;
						break;
					}
				}
			}
		}

		return result;
	}

	function handleRestart() {
		priorAttempt = ['', '', '', ''];
		priorAttemptCorrect = ['', '', '', ''];
		currentAttempt = ['', '', '', ''];
		attempts = 0;
		secretNumber = generateRandomFourDigitString();
		const firstInput = document.getElementById('input-0');
		if (firstInput) {
			firstInput.focus();
		}
	}
</script>

<!-- Debugging info: -->
<!-- <div class="bg-slate-900 text-slate-50">
	<p>debugging</p>
	<p>priorAttempt: {priorAttempt}</p>
	<p>currentAttempt: {currentAttempt}</p>
	<p>attempts: {attempts}</p>
	<p>maxAttempts: {maxAttempts}</p>
	<p>secretNumber: {secretNumber}</p>
	<p>priorAttemptCorrect: {priorAttemptCorrect}</p>
</div> -->

<!-- Legend -->
<div class="fixed top-0 left-0 m-4 rounded-md bg-slate-800 p-2 text-slate-50 shadow-lg">
	<div class="mb-2 flex items-center">
		<div class="mr-2 h-4 w-4 rounded-full bg-sky-500"></div>
		<span>Correct</span>
	</div>
	<div class="mb-2 flex items-center">
		<div class="mr-2 h-4 w-4 rounded-full bg-lime-300"></div>
		<span>Wrong position</span>
	</div>
	<div class="flex items-center">
		<div class="mr-2 h-4 w-4 rounded-full bg-red-400"></div>
		<span>Not in code</span>
	</div>
</div>

<div class="flex min-h-screen flex-col items-center justify-center space-y-4 p-4">
	<!-- Prior Attempt Display -->
	<div class="flex space-x-2">
		{#each priorAttempt as digit, i}
			<div
				class="flex h-16 w-16 items-center justify-center rounded-md border-2 border-slate-400 bg-gray-800 text-xl font-bold text-{priorAttemptColors[
					i
				]}"
			>
				{digit ?? ''}
			</div>
		{/each}
	</div>

	<!-- Current Attempt Input -->
	<div class="flex space-x-2">
		{#each currentAttempt as digit, i}
			<input
				id="input-{i}"
				type="text"
				readonly={true}
				inputmode="numeric"
				pattern="[0-9]*"
				maxlength="1"
				value={digit}
				class="border-slate-20000 h-16 w-16 rounded-md border-2 bg-slate-800 text-center text-xl font-bold text-slate-50 focus:ring-3 focus:ring-slate-50"
			/>
		{/each}
	</div>

	<!-- Attempts Bar -->
	<div class="flex space-x-1">
		{#each Array(maxAttempts) as _, i}
			<div class={`h-12 w-6 rounded-xs ${i < attempts ? 'bg-red-500' : 'bg-slate-400'} `}></div>
		{/each}
	</div>

	<!-- Number Pad -->
	<div class="grid grid-cols-4 gap-2">
		{#each [undefined, 7, 8, 9] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-20 w-20 items-center justify-center rounded-md bg-slate-800 text-xl font-bold text-slate-50 focus:bg-slate-700"
			>
				{number}
			</button>
		{/each}

		{#each [undefined, 4, 5, 6] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-20 w-20 items-center justify-center rounded-md bg-slate-800 text-xl font-bold text-slate-50 focus:bg-slate-700"
			>
				{number}
			</button>
		{/each}
		{#each [0, 1, 2, 3] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-20 w-20 items-center justify-center rounded-md bg-slate-800 text-xl font-bold text-slate-50 focus:bg-slate-700"
			>
				{number}
			</button>
		{/each}
	</div>

	{#if !gameOver}
		<button
			id="submit-button"
			onclick={handleSubmit}
			class="mt-4 rounded-sm bg-slate-800 px-4 py-2 text-slate-50 focus:ring-2 focus:ring-slate-50"
			disabled={!currentAttempt.every((val) => val !== '')}
		>
			Submit
		</button>
	{:else}
		<button
			id="restart-button"
			onclick={handleRestart}
			class="mt-4 rounded-sm bg-slate-800 px-4 py-2 text-slate-50 ring-3 ring-slate-50"
		>
			Restart
		</button>
	{/if}
</div>
<div class="hidden">
	<div class="text-lime-300">text</div>
	<div class="text-red-400">text</div>
	<div class="text-sky-500">text</div>
</div>
