<script lang="ts">
	
	let previousGuesses = [];
	let guess = "";
	const mapping = {};
	const fakeWord = "world";
	const boardSizeX = fakeWord.length;

	let currentGuessAttempt = 0;
	const totalGuessAttempt = 6;

	function onSubmit(e) {
		e.preventDefault();
		console.log(guess);
		
		if (guess === fakeWord) {
			alert("you are a winner")
			// TODO: fire off some confetti
			return
		}

		previousGuesses.push(guess);
		currentGuessAttempt++;
		if (currentGuessAttempt === totalGuessAttempt) {
			alert("you lost!")
			return
		}
		
		const letters = guess.split("")
		
		letters.forEach((letter, i) => {
			// yellow if letter is present in word
			const isPresent = fakeWord.includes(letter);
			// green if letter is present AND in correct position
			if (isPresent) {
				const isCorrectPosition = fakeWord.charAt(i) === letter
				
				if (isCorrectPosition) {
					mapping[letter] = "green"
					return
				}
				mapping[letter] = "yellow"
			}
			mapping[letter] = "grey"
		});
		
	}

</script>

<main>
	<form on:submit={onSubmit}>
		<p>guesses remaining: {totalGuessAttempt - currentGuessAttempt}</p>
		<input type="text" bind:value={guess} />
		<div class="grid">
			{#each Array(totalGuessAttempt) as _, row (row)}
				{#each Array(boardSizeX) as _, col (col)}
				<!-- TODO: fill the characters on the board -->
					{#if row === currentGuessAttempt && col < guess.length && guess}
						<div class="grid-item">{guess[col]}</div>
					{:else}
						<div class="grid-item"></div>
					{/if}
				{/each}
			{/each}
		</div>
	</form>
</main>

<style>

	.grid {
		display: grid;
		grid-template-columns: repeat(5, auto);
		grid-template-rows: repeat(5, auto);
		grid-gap: 20px;
		align-items: center;
		justify-content: center;
	}
	.grid-item {
		border-radius: 5px;
		border: 2px solid grey;
		font-weight: bold;
		font-size: 18px;
		max-width: 100px;
		max-height: 100px;
		text-align: center;
		justify-self: center;
		padding: 20px;
		text-transform: uppercase;
	}
</style>