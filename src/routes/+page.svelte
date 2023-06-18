<script lang="ts">
	import Door from '$lib/components/door.svelte'

	let doorCount = 3
	let carIndex: number
	$: carIndex = Math.floor(Math.random() * doorCount)
</script>

<main class="container">
	<section class="controls">
		<label for="doorCount">Door count: </label>
		<input id="doorCount" bind:value={doorCount} type="number" min="0" />
		<button on:click={() => doorCount--}>-</button>
		<button on:click={() => doorCount++}>+</button>
	</section>
	<section class="doors">
		{#each { length: doorCount } as _, i}
			<Door index={i} isCar={i === carIndex} />
		{/each}
	</section>
</main>

<style lang="scss">
	.controls {
		display: flex;
		align-items: center;
		gap: var(--spacing);

		label {
			flex-shrink: 0;
		}

		:is(input, button) {
			margin: 0;
		}

		button {
			flex-basis: 25%;
		}
	}

	.doors {
		display: grid;
		grid-template-columns: repeat(auto-fit, 100px);
		grid-auto-rows: 200px;
		justify-content: space-evenly;
	}
</style>
