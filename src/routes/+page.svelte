<script lang="ts">
	const maxAttempts = 9;
	const secretNumber = generateRandomFourDigitString();

	let priorAttempt = $state(['', '', '', '']);
	let currentAttempt = $state(['', '', '', '']);
	let attempts = $state(0);

	let remainingAttempts = $derived(maxAttempts - attempts);

	function handleInput(index: number, value: string) {
		console.log('handle input', index, value);
		// need to check if number here
		if (value) {
			currentAttempt[index] = value.slice(-1);
		}
		if (value && index < 3) {
			const nextInput = document.getElementById(`input-${index + 1}`);
			if (nextInput) {
				nextInput.focus();
			}
		}
	}

	function handleNumberPad(number: number | undefined) {
		console.log('handle number pad', number);
		if (number === undefined) return;
		const emptyIndex = currentAttempt.findIndex((val) => val === '');
		if (emptyIndex !== -1) {
			handleInput(emptyIndex, number.toString());
		}
	}

	function handleSubmit() {
		const firstInput = document.getElementById('input-0');
		if (!firstInput) return;
		if (currentAttempt.every((val) => val !== '')) {
			priorAttempt = [...currentAttempt];
			currentAttempt = ['', '', '', ''];
			attempts++;
			firstInput.focus();
		}
	}

	function generateRandomFourDigitString() {
		let result = '';
		for (let i = 0; i < 4; i++) {
			const randomDigit = Math.floor(Math.random() * 10); // Generates a random number between 0 and 9
			result += randomDigit.toString();
		}
		return result;
	}
</script>

<p>debugging</p>
<p>priorAttempt: {priorAttempt}</p>
<p>currentAttempt: {currentAttempt}</p>
<p>attempts: {attempts}</p>
<p>maxAttempts: {maxAttempts}</p>
<p>secretNumber: {secretNumber}</p>
<div class="flex flex-col items-center space-y-4 p-4">
	<!-- Prior Attempt Display -->
	<div class="flex space-x-2">
		{#each priorAttempt as digit}
			<div
				class="flex h-12 w-12 items-center justify-center border-2 border-gray-300 text-xl font-bold"
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
				inputmode="numeric"
				pattern="[0-9]*"
				maxlength="1"
				value={digit}
				oninput={(e) => e.target && handleInput(i, (e.target as HTMLInputElement).value)}
				class="h-12 w-12 border-2 border-blue-500 text-center text-xl font-bold"
			/>
		{/each}
	</div>

	<!-- Health Bar -->
	<div class="flex space-x-1">
		{#each Array(maxAttempts) as _, i}
			<div class={`h-6 w-6 ${i < remainingAttempts ? 'bg-green-500' : 'bg-red-500'}`}></div>
		{/each}
	</div>

	<!-- Number Pad -->
	<div class="grid grid-cols-4 gap-2">
		{#each [undefined, 1, 2, 3] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-16 w-16 items-center justify-center rounded bg-gray-200 text-xl font-bold"
			>
				{number}
			</button>
		{/each}

		{#each [undefined, 4, 5, 6] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-16 w-16 items-center justify-center rounded bg-gray-200 text-xl font-bold"
			>
				{number}
			</button>
		{/each}
		{#each [0, 7, 8, 9] as number}
			<button
				onclick={() => handleNumberPad(number)}
				class="flex h-16 w-16 items-center justify-center rounded bg-gray-200 text-xl font-bold"
			>
				{number}
			</button>
		{/each}
	</div>

	<!-- Submit Button -->
	<button
		onclick={handleSubmit}
		class="mt-4 rounded bg-blue-500 px-4 py-2 text-white"
		disabled={!currentAttempt.every((val) => val !== '')}
	>
		Submit
	</button>
</div>
