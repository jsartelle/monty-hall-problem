<script lang="ts">
	import { browser, dev } from '$app/environment'
	import Door from '$lib/components/door.svelte'

	interface GameRecord {
		carIndex: number
		firstPick: number
		revealedDoor: number
		secondPick: number
	}
	const STORAGE_KEY = 'montyhall_records'

	let gameInfo: {
		doorCount: number
		records: GameRecord[]
	} = {
		doorCount: 3,
		records: [],
	}
	let cheatMode = false

	if (browser && localStorage.getItem(STORAGE_KEY)) {
		gameInfo = JSON.parse(localStorage.getItem(STORAGE_KEY)!)
	}

	/* Setup */

	function changeDoorCount() {
		const answer = prompt(
			'Enter a number between 3 and 100. Changing the door count will clear your records!',
			String(gameInfo.doorCount),
		)

		if (answer === null || answer === '') return
		const parsed = parseInt(answer)

		if (isNaN(parsed)) return alert('Invalid door count!')
		if (parsed < 3) return alert('Enter a number 3 or higher!')
		if (parsed > 100) return alert('Enter a number 100 or lower!')
		if (parsed !== gameInfo.doorCount) {
			gameInfo.doorCount = parsed
			resetRecords(true)
			startGame()
		}
	}

	/** Pick a random door index */
	function randomDoor() {
		return Math.floor(Math.random() * gameInfo.doorCount)
	}

	/* Game Loop */

	let carIndex = 0
	let firstPick: number | null = null
	let revealedDoor: number | null = null
	let secondPick: number | null = null
	let gameOver = false
	let instructions = ''

	function startGame() {
		carIndex = randomDoor()
		firstPick = null
		revealedDoor = null
		secondPick = null
		gameOver = false
		instructions = 'Pick a door!'
	}
	startGame()

	function doorClicked(index: number) {
		document.querySelector('.instructions')?.scrollIntoView()

		if (gameOver) {
			startGame()
			return
		}

		if (firstPick === null) {
			firstPick = index

			while (revealedDoor === null) {
				// Reveal a door that isn't the one you picked, or the one with the car
				const rand = randomDoor()
				if (rand !== firstPick && rand !== carIndex) revealedDoor = rand
			}

			instructions = `You picked door #${firstPick + 1}! Behind door #${
				revealedDoor! + 1
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

	function autoPlay() {
		const answer = prompt('How many rounds? (between 1 and 1000)')

		if (answer === null || answer === '') return
		const parsed = parseInt(answer)

		if (isNaN(parsed)) return alert('Invalid number!')
		if (parsed < 1) return alert('Enter a number 1 or higher!')
		if (parsed > 1000) return alert('Enter a number 1000 or lower!')

		for (let i = 0; i < parsed; i++) {
			while (!gameOver) {
				let door: number
				// don't want to pick the one that's been revealed
				do {
					door = randomDoor()
				} while (revealedDoor && door === revealedDoor)
				doorClicked(door)
			}
			startGame()
		}
	}

	/* Records */

	function updateRecords() {
		gameInfo.records.push({
			carIndex,
			firstPick: firstPick!,
			revealedDoor: revealedDoor!,
			secondPick: secondPick!,
		})
		gameInfo = gameInfo
		localStorage.setItem(STORAGE_KEY, JSON.stringify(gameInfo))
	}

	function resetRecords(skipConfirm = false) {
		if (!skipConfirm) {
			const confirmed = confirm('Are you sure you want to delete all records?')
			if (!confirmed) return
		}
		gameInfo = {
			doorCount: gameInfo.doorCount,
			records: [],
		}
		localStorage.setItem(STORAGE_KEY, JSON.stringify(gameInfo))
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
		winCount = gameInfo.records.filter((record) => record.secondPick === record.carIndex).length
		changedCount = gameInfo.records.filter(
			(record) => record.firstPick !== record.secondPick,
		).length
		changedWinCount = gameInfo.records.filter(
			(record) => record.secondPick === record.carIndex && record.firstPick !== record.secondPick,
		).length
		unchangedCount = gameInfo.records.length - changedCount
		unchangedWinCount = winCount - changedWinCount
	}
</script>

<main class="container">
	<section>
		<h1>Doors</h1>

		<p class="instructions">{instructions}</p>

		<div class="doors">
			{#each { length: gameInfo.doorCount } as _, i}
				<Door
					index={i}
					isCar={i === carIndex}
					revealed={gameOver || revealedDoor === i}
					{cheatMode}
					on:click={() => doorClicked(i)}
				/>
			{/each}
		</div>

		{#if cheatMode}
			<code>
				carIndex: {carIndex}, firstPick: {firstPick}, revealedDoor: {revealedDoor}, secondPick: {secondPick}
			</code>
		{/if}
	</section>
	<section>
		<h1>Records</h1>
		{#if gameInfo.records.length > 0}
			<section>
				<p>
					Wins: {winCount}/{gameInfo.records.length} ({formatPercent(
						winCount,
						gameInfo.records.length,
					)}%)
				</p>
				<p>
					When you didn't change your answer you won {unchangedWinCount}/{unchangedCount} times. ({formatPercent(
						unchangedWinCount,
						unchangedCount,
					)}%)
				</p>
				<p>
					When you changed your answer you won {changedWinCount}/{changedCount} times. ({formatPercent(
						changedWinCount,
						changedCount,
					)}%)
				</p>
			</section>
			<section class="records">
				<table role="grid">
					<thead>
						<tr>
							<th scope="col">#</th>
							<th scope="col">First Pick</th>
							<th scope="col">Revealed</th>
							<th scope="col">Second Pick</th>
							<th scope="col">Winning Door</th>
							<th scope="col">Prize</th>
						</tr>
					</thead>
					<tbody>
						{#each gameInfo.records as { carIndex, firstPick, revealedDoor, secondPick }, index}
							<tr>
								<th scope="row">{index + 1}</th>
								<td>{firstPick + 1}</td>
								<td>{revealedDoor + 1}</td>
								<td>{secondPick + 1}</td>
								<td>{carIndex + 1}</td>
								<td>{secondPick === carIndex ? '🚗' : '🐐'}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</section>
		{/if}
		<div class="controls">
			<button on:click={autoPlay}>Auto Play</button>
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
		scroll-margin-top: var(--spacing);
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

	.records {
		max-height: 500px;
		overflow-y: scroll;
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
