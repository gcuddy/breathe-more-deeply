<script lang="ts">
	import { fly } from 'svelte/transition';

	let lastX = $state(0);
	let lastY = $state(0);
	let lastCheckedX = $state(0);
	let lastCheckedY = $state(0);
	let checking = $state(false);

	let startingTop = Math.random() * 90;
	let startingLeft = Math.random() * 90;

	let endingTop = Math.random() * 90;
	let endingLeft = Math.random() * 90;

	let status: 'playing' | 'won' | 'lost' | 'unstarted' = $state('unstarted');

	let movingTooFast = $state(false);
	let movingTooFastIncrementer = $state(0);
	let shouldShowWarning = $derived(movingTooFastIncrementer > 0);

	function check(x: number, y: number) {
		console.log('checking', x, y);
		checking = true;
		if (x === lastCheckedX && y === lastCheckedY) {
			console.log('not moving');
			status = 'lost';
		}
		lastCheckedX = x;
		lastCheckedY = y;
		checking = false;
	}

	function handleMouseMove(event: MouseEvent) {
		if (status !== 'playing') {
			return;
		}
		lastX = event.clientX;
		lastY = event.clientY;
		if (
			event.movementX > 1 ||
			event.movementY > 1 ||
			event.movementX < -1 ||
			event.movementY < -1
		) {
			console.log('fast');
			if (movingTooFast) {
				movingTooFastIncrementer++;
				if (movingTooFastIncrementer > 25) {
					status = 'lost';
				}
			}
			movingTooFast = true;
		} else {
			console.log('slow');
			movingTooFastIncrementer = Math.max(0, movingTooFastIncrementer - 1);
			console.log(movingTooFastIncrementer);
			if (movingTooFastIncrementer === 0) {
				movingTooFast = false;
			}
		}
	}

	setInterval(() => {
		if (checking || status !== 'playing') {
			return;
		}
		check(lastX, lastY);
	}, 2000);
</script>

{#if status === 'lost' || status === 'won'}
	<div class="overlay">
		<span> {status.toUpperCase()}! </span>
	</div>
{/if}
{#if shouldShowWarning}
	<div
		in:fly={{
			delay: 50
		}}
		class="warning"
	>
		Warning: Slow Down! Breathe More Deeply!
	</div>
{/if}
<div id="game" role="none" on:mousemove={handleMouseMove}>
	<div
		role="none"
		on:mouseover={() => {
			if (status === 'unstarted') {
				status = 'playing';
			}
		}}
		on:focus
		class="starting"
		style:top="{startingTop}%"
		style:left="{startingLeft}%"
	>
		<svg
			xmlns="http://www.w3.org/2000/svg"
			width="24"
			height="24"
			viewBox="0 0 24 24"
			fill="none"
			stroke="currentColor"
			stroke-width="2"
			stroke-linecap="round"
			stroke-linejoin="round"
			class="lucide lucide-home"
			><path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z" /><polyline
				points="9 22 9 12 15 12 15 22"
			/></svg
		>
	</div>
	<div
		role="none"
		on:mouseover={() => {
			if (status === 'playing') {
				status = 'won';
			}
		}}
		on:focus
		class="ending"
		style:top="{endingTop}%"
		style:left="{endingLeft}%"
	>
		<svg
			xmlns="http://www.w3.org/2000/svg"
			width="24"
			height="24"
			viewBox="0 0 24 24"
			fill="none"
			stroke="currentColor"
			stroke-width="2"
			stroke-linecap="round"
			stroke-linejoin="round"
			class="lucide lucide-tree-deciduous"
			><path
				d="M8 19a4 4 0 0 1-2.24-7.32A3.5 3.5 0 0 1 9 6.03V6a3 3 0 1 1 6 0v.04a3.5 3.5 0 0 1 3.24 5.65A4 4 0 0 1 16 19Z"
			/><path d="M12 19v3" /></svg
		>
	</div>
	<div class="ending"></div>
	<span>
		{status}
	</span>
</div>

<style>
	:global(body) {
		margin: 0;
	}
	div#game {
		width: 100vw;
		height: 100vh;
		background-color: lightcyan;
		border: 1px solid black;
	}
	.warning {
		/* center  */
		position: absolute;
		top: 1rem;
	}

	.overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: 100vh;
		background-color: #000000;
		font-size: 5rem;
		z-index: 2;
		color: white;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.starting,
	.ending {
		display: inline-block;
		position: fixed;
	}

	.starting svg {
		height: 100px;
		width: 100px;
		fill: brown;
	}
	.ending svg {
		height: 100px;
		width: 100px;
		fill: green;
	}
</style>
