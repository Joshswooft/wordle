<script lang="ts">
	
	let guess = "";
	const colorMapping = {};
	const boardList = [];
	// TODO: replace fake word with API call
	const fakeWord = "world";
	const boardSizeX = fakeWord.length;

	let currentGuessAttempt = 0;
	const totalGuessAttempt = 6;

	let hasWon = false;

	for(let i = 0; i < boardSizeX * totalGuessAttempt; i++) {
		boardList.push("")
	}

	function onChange() {
		guess = guess.toLocaleLowerCase();
	}


	function onSubmit(e) {
		e.preventDefault();
		console.log(guess);

		guess = guess.toLocaleLowerCase();

		if (guess.length !== boardSizeX) {
			return
		}

		// TODO: check word is valid and if not then dont contribute as guess attempt
		
		if (guess === fakeWord) {
			alert(`You correctly guessed the word in ${currentGuessAttempt + 1} tries!`)
			// TODO: fire off some confetti
			hasWon = true;
		}
		
		guess.split("").forEach((letter, index) => {
			console.log(index + (currentGuessAttempt * boardSizeX));
			boardList[index + (currentGuessAttempt * boardSizeX)] = letter;
		})
		currentGuessAttempt++;
		if (currentGuessAttempt === totalGuessAttempt && !hasWon) {
			alert("The word was " + fakeWord)
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
					colorMapping[letter + i] = "green"
					return
				}
				colorMapping[letter + i] = "yellow"
				return
			}
			colorMapping[letter + i] = "grey"
		});
		guess = "";
	}

</script>

<main>
	<form on:submit={onSubmit}>
		<p>guesses remaining: {totalGuessAttempt - currentGuessAttempt}</p>
		<input type="text" on:change={onChange} bind:value={guess} maxlength={boardSizeX} />
		<div class="grid">
			{#each boardList as item, index}
				<div class={`grid-item ${colorMapping[item + (index % boardSizeX)]}`}>{item}</div>
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

	.grey, .yellow, .green {
		color: white;
		border: none;
	}
	.grey {
		background-color: gray;
	}
	.yellow {
		background-color: #cfcf27;
	}
	.green {
		background-color: green;
	}

</style>