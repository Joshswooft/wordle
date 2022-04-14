<script>
    import { querystring } from 'svelte-spa-router'
    let searchParams = new URLSearchParams($querystring);
    let correctWord = searchParams.get("correctWord");
    let guessedWord = searchParams.get("guessedWord");
</script>
<main>
    <div class="center">
        <h1>You lose!</h1>
        <p>The correct word was <span>"{correctWord}"</span>, but you guessed <span>"{guessedWord}"</span></p>
    </div>
</main>

<style>
    .center {
        text-align: center;
        margin: auto;
        margin-top: 50px;
        padding: 10px;
    }

    span {
        font-weight: bold;
        text-align: center;
    }
</style>