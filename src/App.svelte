<script>
	import {tick} from 'svelte'
	let face = 'normal'
	let x = 0
	let y = 0
	let makeItHarderMode = false
	
	let [cherryX, cherryY] = getRandomCherryPosition()
	const grid = 20
	const width = 600
	const height = 300
	const safeZone = 200
	let ms = 800
	const msMax = 2000
	const msMin = 100
	$: ms = Math.min(ms, msMax)
	$: ms = Math.max(ms, msMin)
	let chasing = false
	let points = 0
	let caught = false
	let buttonDown = false;
	let buttonUp = true;
	let baddieCount = 3;
	let oldBaddieCount = 3;
	let baddies = makeBaddies(baddieCount)
	$: if (baddieCount) {
		if (baddieCount > oldBaddieCount) {
			const newBaddies = makeBaddies(baddieCount - oldBaddieCount)
			newBaddies.forEach(baddie => baddies.push(baddie))
			baddies = baddies
		} else if (baddieCount < oldBaddieCount) {
			const killBaddiesCount = oldBaddieCount - baddieCount
			console.log(`Killing ${killBaddiesCount} baddies`)
			for (let i = 0; i < killBaddiesCount; i++) {
				baddies.pop()
				baddies = baddies
			}
		}
		oldBaddieCount = baddieCount
	}
	
	$: x = Math.min(x, width - grid)
	$: y = Math.min(y, height - grid)
	$: x = Math.max(x, 0)
	$: y = Math.max(y, 0)
	
	// When you get the cherry, move the cherry to a new position and add points.
	$: if (x === cherryX && y === cherryY) {
		[cherryX, cherryY] = getRandomCherryPosition(true)
		points = points + calculatePointValue()
		
		if (makeItHarderMode) {
			baddieCount++
			ms = Math.floor(0.9 * ms)
		}
	}
	
	$: if (y || x) {buttonDown = true; buttonUp = false}
	
	$: for (const [baddieX, baddieY, face] of baddies) {
			if (x === baddieX && y === baddieY) {
				chasing = false
				caught = true
				alert('You were caught by ' + face)
			}
	}
	
	
	function position(x, y) {
		return `
			bottom: ${y}px;
			left: ${x}px
		`
	}
	
	function calculatePointValue() {
		return baddieCount * (msMax - ms) + 1
	}
	
	function reset() {
		chasing = false
		caught = false
		baddies = makeBaddies(baddieCount)
		x = 0
		y = 0
		const cherryPositions = getRandomCherryPosition()
		cherryX = cherryPositions[0]
		cherryY = cherryPositions[1]
		points = 0
	}
	
	function makeBaddies(baddieCount) {
		const baddies = [];
		for (let i = 0; i < baddieCount; i++) {
			baddies.push(getRandomBaddie(getRandomBaddieEmoji()))
		}
		
		return baddies;
	}
	
	function getRandomBaddieEmoji() {
		const emoji = ['😈', '🦁', '🤡', '👽', '👀', '👏🏻', '💩', '👾', '👻', '👿', '🎃', '🤖', '👺', '🤮', '☠️', '🐸', '🐍']
		
		return emoji[Math.floor(Math.random() * emoji.length)]
	}
	
	function getRandomInt(min, max) {
		min = Math.ceil(min);
		max = Math.floor(max);
  	return Math.floor(Math.random() * (max - min) + min);
	}
	
	function getRandomBaddie(face) {
		let x, y
		do {
			x = getRandomInt(0, width / grid) * grid
			y = getRandomInt(0, height / grid) * grid
		} while( x <= safeZone && y <= safeZone)
		
		return [x, y, face]
	}
	
	function getRandomCherryPosition(anywhere = false) {
		const x = getRandomInt((anywhere ? 0 : (width - safeZone)) / grid, width / grid) * grid
		const y = getRandomInt((anywhere ? 0 : (height - safeZone)) / grid, height / grid) * grid
		
		return [x, y]
	}
	
	
	function pressKey({ key }) {
		if (caught || (buttonDown && !buttonUp)) {
			return
		}
		if (key === 'ArrowUp') {
			y = y + grid
		} else if (key === 'ArrowDown') {
			y = y - grid
		} else if (key === 'ArrowRight') {
			x = x + grid
		} else if (key === 'ArrowLeft') {
			x = x - grid
		}
		chasing = true
	}
	
	function releaseKey({ key }) {
		if (['ArrowDown', 'ArrowUp', 'ArrowLeft', 'ArrowRight'].includes(key)) {
			buttonUp = true;
		}
	}
	
	async function chase() {
		if (!chasing) {
			return
		}
		baddieMoves:
		for (const baddieNumber in baddies) {
			let [baddieX, baddieY, face] = baddies[baddieNumber]
			const xDiff = Math.abs(baddieX - x)
			const yDiff = Math.abs(baddieY - y)
			if (xDiff > yDiff) {
				if (x < baddieX) {
					baddieX = baddieX - grid
				} else if (x > baddieX) {
					baddieX = baddieX + grid
				}
			} else {
				if (y < baddieY) {
					baddieY = baddieY - grid
				} else if (y > baddieY) {
					baddieY = baddieY + grid
				}
			}
			if (baddieX === cherryX && baddieY === cherryY) {
				continue baddieMoves;
			}
			
			for (const [otherBaddieX, otherBaddieY] of baddies) {
				if (baddieX === otherBaddieX && baddieY === otherBaddieY) {
					continue baddieMoves;
				}
			}
			baddies[baddieNumber] = [baddieX, baddieY, face]
		}
	}

	let clear
	$: {
		clearInterval(clear)
		clear = setInterval(chase, ms)
	}
</script>
<style>
	:global(body) {
		overflow: hidden;
	}
	main {
		width: 600px;
		height: 300px;
		background: #eee;
		position: relative;
	}
	
	b {
		z-index: 2;
		display: block;
		position: absolute;
	}
	
	i {
		display: block;
		position: absolute;
		background: #e6e5e6;
		width: 200px;
		height: 200px;
	}
	
	i.safe {
		bottom: 0;
		left: 0;
	}
	
	i.end {
		top: 0;
		right: 0;
		background: #e9f5e9
	}
	
	p.speed, p.baddies {
		display: flex;
		align-items: center;
	}
	
	p.speed input, p.baddies input {
		margin-left: 1rem;
		top: 0.35rem;
		position: relative;
	}
	
	p.speed input {
		direction: rtl;
	}
	
	p.speed span, p.baddies span {
		display: block;
		margin-left: 0.5rem;
	}
</style>

<svelte:window on:keydown={pressKey} on:keyup={releaseKey} />
<p>
	{points} points
</p>
<main>
	<i class="safe"></i>
	<i class="end"></i>
	<b style={position(x, y)}>
		🙂
	</b>
	
	{#each baddies as [x, y, face]}
		<b style={position(x, y)}>
		{face}
		</b>
	{/each}
	
	<b style={position(cherryX, cherryY)}>
		🍒
	</b>
</main>
<p>
	<input type="checkbox" bind:checked={chasing} /> Chasing<br />
	<input type="checkbox" bind:checked={caught} /> Caught
</p>
<hr />
<p>
	<input type="checkbox" bind:value={makeItHarderMode} /> Make it harder every time you get a cherry
</p>
<p class="speed">
	Speed: <input type="range" min={msMin} max={msMax} bind:value={ms} class="slider" id="myRange"><span><span>{ms}ms baddie delay</span>
</p>
<p class="baddies">
	Baddies: <input type="range" min=1 max=100 bind:value={baddieCount} class="slider" id="myRange"><span>{baddieCount}</span>
</p>
<p>
	<button on:click={reset}>
		Reset
	</button>
</p>