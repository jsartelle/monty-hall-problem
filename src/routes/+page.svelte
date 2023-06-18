<script lang="ts">
	import { browser, dev } from '$app/environment'
	import Door from '$lib/components/door.svelte'

	interface GameRecord {
		carIndex: number
		firstPick: number
		openedDoor: number
		secondPick: number
	}
	const STORAGE_KEY = 'montyhall_records'

	let records: GameRecord[] = []
	if (browser && localStorage.getItem(STORAGE_KEY)) {
		records = JSON.parse(localStorage.getItem(STORAGE_KEY)!)
	}

	/* Setup */

	let doorCount = 3
	let cheatMode = false

	function changeDoorCount() {
		const answer = prompt(
			'Enter a number between 3 and 100. Changing the door count will clear your records!',
			String(doorCount),
		)

		if (answer === null || answer === '') return
		const parsed = parseInt(answer)

		if (isNaN(parsed)) return alert('Invalid door count!')
		if (parsed < 3) return alert('Pick a number higher than 2!')
		if (parsed > 100) return alert('Pick a lower number!')
		if (parsed !== doorCount) {
			doorCount = parsed
			resetRecords(true)
			startGame()
		}
	}

	/** Pick a random door index */
	function pickRandomDoor() {
		return Math.floor(Math.random() * doorCount)
	}

	/* Game Loop */

	let carIndex = 0
	let firstPick: number | null = null
	let openedDoor: number | null = null
	let secondPick: number | null = null
	let gameOver = false
	let instructions = ''

	function startGame() {
		carIndex = pickRandomDoor()
		firstPick = null
		openedDoor = null
		secondPick = null
		gameOver = false
		instructions = 'Pick a door!'
	}
	startGame()

	function doorClicked(index: number) {
		if (gameOver) {
			startGame()
			return
		}

		if (firstPick === null) {
			firstPick = index

			while (openedDoor === null) {
				// Reveal a door that isn't the one you picked, or the one with the car
				const rand = pickRandomDoor()
				if (rand !== firstPick && rand !== carIndex) openedDoor = rand
			}

			instructions = `You picked door #${firstPick + 1}! Behind door #${
				openedDoor! + 1
			} is a goat.\nStick with your choice, or change doors?`
		} else if (secondPick === null) {
			secondPick = index

			if (secondPick === carIndex) {
				instructions = 'Congratulations, you picked the car!\nClick any door to restart.'
			} else {
				instructions = `Sorry, the car was behind Door #${
					carIndex + 1
				}!\nClick any door to try again.`
			}

			updateRecords()
			gameOver = true
			return
		}
	}

	/* Records */

	function updateRecords() {
		records.push({
			carIndex,
			firstPick: firstPick!,
			openedDoor: openedDoor!,
			secondPick: secondPick!,
		})
		records = records
		localStorage.setItem(STORAGE_KEY, JSON.stringify(records))
	}

	function resetRecords(skipConfirm = false) {
		if (!skipConfirm) {
			const confirmed = confirm('Are you sure you want to delete all records?')
			if (!confirmed) return
		}
		records = []
		localStorage.removeItem(STORAGE_KEY)
	}

	function formatPercent(divisor: number, dividend: number) {
		const result = parseFloat(((divisor / dividend) * 100).toFixed(2))
		return isNaN(result) ? 0 : result
	}

	let winCount: number
	let changedCount: number
	let changedWinCount: number
	let unchangedCount: number
	let unchangedWinCount: number
	$: {
		winCount = records.filter((record) => record.secondPick === record.carIndex).length
		changedCount = records.filter((record) => record.firstPick !== record.secondPick).length
		changedWinCount = records.filter(
			(record) => record.secondPick === record.carIndex && record.firstPick !== record.secondPick,
		).length
		unchangedCount = records.length - changedCount
		unchangedWinCount = winCount - changedWinCount
	}
</script>

<main class="container">
	<section>
		<h1>Doors</h1>

		<p class="instructions">{instructions}</p>

		<div class="doors">
			{#each { length: doorCount } as _, i}
				<Door
					index={i}
					isCar={i === carIndex}
					revealed={gameOver || openedDoor === i}
					{cheatMode}
					on:click={() => doorClicked(i)}
				/>
			{/each}
		</div>

		{#if cheatMode}
			<code>
				carIndex: {carIndex}, firstPick: {firstPick}, openedDoor: {openedDoor}, secondPick: {secondPick}
			</code>
		{/if}
	</section>
	<section>
		<h1>Records</h1>
		{#if records.length > 0}
			<section>
				<p>
					Wins: {winCount}/{records.length} ({formatPercent(winCount, records.length)}%)
				</p>
				<p>
					When you changed your answer you won {changedWinCount}/{changedCount} times. ({formatPercent(
						changedWinCount,
						changedCount,
					)}%)
				</p>
				<p>
					When you didn't change your answer you won {unchangedWinCount}/{unchangedCount} times. ({formatPercent(
						unchangedWinCount,
						unchangedCount,
					)}%)
				</p>
			</section>
			<table role="grid">
				<thead>
					<tr>
						<th scope="col">#</th>
						<th scope="col">First Pick</th>
						<th scope="col">Opened</th>
						<th scope="col">Second Pick</th>
						<th scope="col">Winning Door</th>
						<th scope="col">Prize</th>
					</tr>
				</thead>
				<tbody>
					{#each records as { carIndex, firstPick, openedDoor, secondPick }, index}
						<tr>
							<th scope="row">{index + 1}</th>
							<td>{firstPick + 1}</td>
							<td>{openedDoor + 1}</td>
							<td>{secondPick + 1}</td>
							<td>{carIndex + 1}</td>
							<td>{secondPick === carIndex ? '🚗' : '🐐'}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		{/if}
		<div class="controls">
			<button on:click={changeDoorCount}>Change Door Count</button>
			<button on:click={() => resetRecords(false)}>Reset Records</button>
			{#if dev}
				<label>
					<span>Cheat Mode (dev)</span>
					<input bind:checked={cheatMode} type="checkbox" role="switch" />
				</label>
			{/if}
		</div>
	</section>
</main>

<style lang="scss">
	.instructions {
		padding: calc(var(--spacing) / 2);
		white-space: pre-line;
		text-align: center;
		background-color: var(--code-background-color);
		border-radius: var(--border-radius);
	}

	.doors {
		display: grid;
		grid-template-columns: repeat(auto-fit, 100px);
		grid-auto-rows: 200px;
		justify-content: space-evenly;
	}

	.controls {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
		gap: var(--spacing);

		* {
			flex-grow: 1;
		}

		button {
			width: auto;
			margin: 0;
		}

		label {
			cursor: pointer;
		}
	}
</style>
