<script lang="ts">
	import { onMount } from "svelte";
  import { push } from 'svelte-spa-router'
  import { Confetti } from "svelte-confetti"

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

	function onFocus(el) {
		el.target.focus()
	}
	
	let guess = "";
	const colorMapping = {};
  let boardList = [];
	const boardSizeX = fakeWord.length;
	const qwertyRows = "qwertyuiop|asdfghjkl|zxcvbnm".split("|")
	let currentGuessAttempt = 0;
	const totalGuessAttempt = 6;

	let hasWon = false;

	for(let i = 0; i < boardSizeX * totalGuessAttempt; i++) {
		boardList.push("")
	}

	function onChange(e) {
		guess = guess.toLocaleLowerCase();
		if (guess.length > 5) {
			guess = guess.slice(0,5)
		}
		guess.split("").forEach((letter, index) => {
			boardList[index + (currentGuessAttempt * boardSizeX)] = letter;
		})

	}

	async function handleSubmit() {
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
			guess.split("").forEach((letter, index) => {
				boardList[index + (currentGuessAttempt * boardSizeX)] = "";
			})
			guess = ""
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
			hasWon = true;
		}
		
		currentGuessAttempt++;
		if (currentGuessAttempt === totalGuessAttempt && !hasWon) {
			push(`/lost?correctWord=${fakeWord}&guessedWord=${guess}`);
			return
		}
		guess = "";
	}

	async function onSubmit(e) {
		e.preventDefault();
		handleSubmit();
	}

</script>
<main>
  {#if hasWon}
    <div style="
      position: fixed;
      top: -50px;
      left: 0;
      height: 100vh;
      width: 100vw;
      display: flex;
      justify-content: center;
      overflow: hidden;
      pointer-events: none;"
    >
      <Confetti x={[-5, 5]} y={[0, 0.1]} delay={[500, 2000]} infinite duration=5000 amount=200 fallDistance="100vh" />
    </div>

  {/if}
	<form on:submit={onSubmit}>
		{#if guess.length == boardSizeX && handleSubmit()}
		{/if}
		<p>guesses remaining: {totalGuessAttempt - currentGuessAttempt}</p>
		<!-- on:blur forces the focus back onto this input element if you click anywhere else on the page -->
		<input use:init on:blur={onFocus} class="hidden" type="text" on:change={onChange} bind:value={guess} maxlength={boardSizeX} />
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
