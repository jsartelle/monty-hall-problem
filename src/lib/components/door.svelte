<script lang="ts">
	import { dev } from '$app/environment'

	export let index = 0
	export let isCar = false
</script>

<button class="door contrast outline" on:click>
	<p>Door #{index + 1}</p>
	{#if dev}
		<p class="contents">{isCar ? '🚗' : '🐐'}</p>
	{/if}
</button>

<style>
	.door {
		position: relative;
		display: flex;
		flex-direction: column;
		align-items: center;
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
		border: 1px dashed currentColor;
	}
</style>