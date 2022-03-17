<script lang="ts">
	import { onMount } from "svelte";

	let fakeWord = "world";

	async function getWordOfTheDay() {
		const response = await fetch("https://api.frontendeval.com/fake/word");
		return await response.text();
	}

	async function checkWordIsValid(word: string) {
		const response = await fetch("https://api.frontendeval.com/fake/word/valid", {
			body: JSON.stringify({
				word
			}),
			method: "POST",
			headers: {
        		'Content-Type': 'application/json'
    		}
		});

		return await response.json();
	}

	onMount(async () => {
		fakeWord = await getWordOfTheDay();
		
	})

	function init(el){
    	el.focus()
  	}
	
	let guess = "";
	const colorMapping = {};
	const boardList = [];
	const boardSizeX = fakeWord.length;
	const qwertyRows = "qwertyuiop|asdfghjkl|zxcvbnm".split("|")
	let currentGuessAttempt = 0;
	const totalGuessAttempt = 6;

	let hasWon = false;
	

	for(let i = 0; i < boardSizeX * totalGuessAttempt; i++) {
		boardList.push("")
	}

	function onChange() {
		guess = guess.toLocaleLowerCase();
		guess.split("").forEach((letter, index) => {
			boardList[index + (currentGuessAttempt * boardSizeX)] = letter;
		})
	}


	async function onSubmit(e) {
		e.preventDefault();

		// game is over
		if(hasWon || totalGuessAttempt - currentGuessAttempt == 0) {
			return
		}

		guess = guess.toLocaleLowerCase();

		if (guess.length !== boardSizeX) {
			return
		}

		const isValid: boolean = await checkWordIsValid(guess);
		if (!isValid) {
			alert("word is not in the dictionary, try again!")
			return
		}
		
		
		guess.split("").forEach((letter, index) => {
			boardList[index + (currentGuessAttempt * boardSizeX)] = letter;
		})
		
		const letters = guess.split("")
		
		letters.forEach((letter, i) => {
			// yellow if letter is present in word
			const isPresent = fakeWord.includes(letter);
			// green if letter is present AND in correct position
			if (isPresent) {
				const isCorrectPosition = fakeWord.charAt(i) === letter
				
				if (isCorrectPosition) {
					colorMapping[letter + i] = "green"
					// seperate entry for keyboard - probably not best way to do this
					colorMapping[letter] = "green"
					return
				}
				colorMapping[letter + i] = "yellow"
				colorMapping[letter] = "yellow"
				return
			}
			colorMapping[letter + i] = "grey"
			colorMapping[letter] = "grey"
		});
		
		
		if (guess === fakeWord) {
			alert(`You correctly guessed the word in ${currentGuessAttempt + 1} tries!`)
			// TODO: fire off some confetti
			hasWon = true;
		}
		
		currentGuessAttempt++;
		if (currentGuessAttempt === totalGuessAttempt && !hasWon) {
			alert("The word was " + fakeWord)
			return
		}
		guess = "";
	}

</script>

<main>
	<form on:submit={onSubmit}>
		<p>guesses remaining: {totalGuessAttempt - currentGuessAttempt}</p>
		<input use:init class="hidden" type="text" on:change={onChange} bind:value={guess} maxlength={boardSizeX} />
		<div class="grid">
			{#each boardList as item, index}
				{#if parseInt(`${index / boardSizeX}`) == currentGuessAttempt && guess[index % boardSizeX]}
					<div class={`grid-item active`}>{guess[index % boardSizeX]}</div>
				{:else}
					<div class={`grid-item ${colorMapping[item + (index % boardSizeX)]}`}>{item}</div>
				{/if}
			{/each}
		</div>
		<div class="keyboard">
			{#each qwertyRows as qwertyKeys}
				<div class="keyboard-row">
					{#each qwertyKeys.split("") as key}
						<div class={`keyboard-key ${colorMapping[key]}`}>{key}</div>
					{/each}
				</div>
			{/each}
		</div>
	</form>
</main>

<style>

	.hidden {
		opacity: 0;
		width: 0;
		overflow: hidden;
	}
	.grid {
		display: grid;
		grid-template-columns: repeat(5, auto);
		grid-template-rows: repeat(5, auto);
		grid-gap: 8px;
		align-items: center;
		justify-content: center;
	}
	.grid-item {
		display: inline-grid;
		border-radius: 5px;
		border: 2px solid #dee1e9;
		font-weight: bold;
		font-size: 24px;
		height: 64px;
		width: 64px;
		text-align: center;
		justify-self: center;
		padding: 10px;
		text-transform: uppercase;
		box-sizing: border-box;
		align-items: center;
	}

	.grid-item.active {
		border: 2px solid grey;
	}

	.grey, .yellow, .green {
		color: white;
		border: none;
	}
	.grey {
		background-color: rgba(112, 128, 144, 0.644);
	}
	.yellow {
		background-color: #cfcf27;
	}
	.green {
		background-color: green;
	}

	.keyboard {
		display: grid;
		margin: 20px;
	}
	.keyboard-row {
		display: inline-flex;
		justify-content: center;
	}
	.keyboard-key {
		width: 50px;
		height: 40px;
		border-radius: 5px;
		border: 1px solid #dee1e9;
		display: flex;
		text-align: center;
		text-transform: uppercase;
		justify-content: center;
		align-items: center;
	}

</style>