<script lang="ts">
	export let index = 0
	export let isCar = false
	export let revealed = false
	export let cheatMode = false
</script>

<button class="door contrast outline" on:click>
	<p>Door #{index + 1}</p>
	{#if revealed || cheatMode}
		<p class="contents" class:cheat={!revealed && cheatMode}>{isCar ? '🚗' : '🐐'}</p>
	{/if}
</button>

<style>
	.door {
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-between;
		text-align: center;
		border: 1px solid currentColor;

		&::after {
			--size: 10px;
			content: '';
			border: 1px solid currentColor;
			height: var(--size);
			width: var(--size);
			border-radius: 50%;
			position: absolute;
			top: calc(50% - (var(--size) / 2));
			right: 10px;
		}
	}

	.contents {
		font-size: 2em;

		&.cheat {
			opacity: 0.25;
		}
	}
</style>
