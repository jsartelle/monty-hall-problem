<script lang="ts">
	import { dev } from '$app/environment'
	import Door from '$lib/components/door.svelte'

	let doorCount = 3
	let cheatMode = false
	let carIndex: number
	$: carIndex = Math.floor(Math.random() * doorCount)

	function changeDoorCount() {
		const answer = parseInt(
			prompt(
				'Enter a number between 3 and 100. Changing the door count will reset your stats!',
				String(doorCount),
			) as string,
		)
		if (!isNaN(answer)) {
			if (answer < 3) return alert('Pick a number higher than 2!')
			if (answer > 100) return alert('Pick a lower number!')
			if (answer !== doorCount) doorCount = answer
		} else {
			return alert('Invalid door count!')
		}
	}
</script>

<main class="container">
	<section>
		<h1>Doors</h1>
		<div class="doors">
			{#each { length: doorCount } as _, i}
				<Door index={i} isCar={i === carIndex} {cheatMode} />
			{/each}
		</div>
	</section>
	<section>
		<h1>Stats</h1>
		<button on:click={changeDoorCount}>Change Door Count</button>
		{#if dev}
			<label>
				<span>Cheat Mode</span>
				<input bind:checked={cheatMode} type="checkbox" role="switch" />
			</label>
		{/if}
	</section>
</main>

<style lang="scss">
	.doors {
		display: grid;
		grid-template-columns: repeat(auto-fit, 100px);
		grid-auto-rows: 200px;
		justify-content: space-evenly;
	}
</style>
