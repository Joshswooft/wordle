<script lang="ts">
	const boardSizeX = 5;
	const boardSizeY = 5;

</script>

<main>
	<div class="grid">
		{#each Array(boardSizeY) as _, row (row)}
			{#each Array(boardSizeX) as _, col (col)}
				<div class="grid-item">{row}{col}</div>
			{/each}
		{/each}
	</div>
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
	}
</style>