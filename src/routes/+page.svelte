<script lang="ts">
	import { onMount, tick } from 'svelte';
	import imgHitpad from '$lib/images/hitpad@3x.png';
	import Page from './sverdle/how-to-play/+page.svelte';

	const keyA = 65;
	const keyS = 83;
	const keyK = 75;
	const keyL = 76;
	const keySpace = 32;
	let chartCsv: Object = {};

	const graphicSettings = {
		beatDuration: 3, // seconds
		hitpadOffset: [116 / 2, 62 / 2]
	};
	const lanesGraphics = [
		{
			start_xy: [100, 100],
			end_xy: [100, 600]
		},
		{
			start_xy: [300, 100],
			end_xy: [300, 600]
		},
		{
			start_xy: [500, 100],
			end_xy: [500, 600]
		},
		{
			start_xy: [700, 100],
			end_xy: [700, 600]
		}
	];

	let consoleOutputText = '';
	let wallTimeStartedAt = 0;
	let gameTimePausedAt = 0;
	let isGameRunning = false;
	let currentScore = 0;

	// Each lane gets their own id => timings[]
	let beatsStack = {};

	let audioPlayer: HTMLMediaElement;

	onMount(async () => {
		// Load the song, chart, bind
		const chartCsvResponse = await fetch('/track-alpha9-nbts/chart.csv');
		chartCsv = await chartCsvResponse.text();
	});

	// This doesn't start the game
	function setupGame() {
		// Load the beats stack
		beatsStack = parseCSV(chartCsv);

		// Reset game states
		currentScore = 0;
		audioPlayer.fastSeek(0);
		audioPlayer.play();
		// Bind rendering event
	}

	// Set timeout
	// function resumeGame() {}

	// Remove all the event bindings
	// function pauseGame() {}

	// Main control handling
	// - Start, pause, rewind game
	// - Handle beats
	async function onKeyDown(e) {
		if (![keyA, keyS, keyK, keyL, keySpace].includes(e.keyCode)) {
			return;
		}

		e.preventDefault();

		// Space bar starts-pauses the game
		if (e.keyCode == keySpace) {
			if (wallTimeStartedAt == 0 && gameTimePausedAt == 0) {
				// Start game
				wallTimeStartedAt = Date.now();
				audioPlayer.play();
				audioPlayer.fastSeek(0);
				currentScore = 0;
				isGameRunning = true;
				consoleOutputText += `Game started at ${wallTimeStartedAt} <br>`;
			} else if (wallTimeStartedAt != 0 && gameTimePausedAt == 0) {
				// setupGame();
				return;

				// Pause game -- implement below later
				gameTimePausedAt = Date.now() - wallTimeStartedAt;
				audioPlayer.pause();
				isGameRunning = false;
				consoleOutputText += `Game paused at ${gameTimePausedAt} <br>`;
			} else if (wallTimeStartedAt != 0 && gameTimePausedAt != 0) {
				return; // Resume from pause
				consoleOutputText += `Game resumed at ${gameTimePausedAt} <br>`;
				wallTimeStartedAt = Date.now() - gameTimePausedAt;
				gameTimePausedAt = 0;
				audioPlayer.play();
				isGameRunning = true;
			}
		}

		if (isGameRunning && [keyA, keyS, keyK, keyL].includes(e.keyCode)) {
			let game_time = Date.now() - wallTimeStartedAt;
			consoleOutputText += `Key ${e.keyCode} hit at ${game_time} ms <br>`;
		}

		await tick();
	}

	// Utilities
	// Simple CSV Parser specific to your data structure
	function parseCSV(csv) {
		let lines = csv.split('\n');
		let result = {};
		// Skip header line, iterate from second line
		for (let i = 1; i < lines.length; i++) {
			if (!lines[i]) continue;
			const [strikeAt, lane] = lines[i].split(',');
			const key = `lane_${lane.trim()}`;
			if (!result[key]) {
				result[key] = [];
			}
			result[key].push(parseFloat(strikeAt.trim()));
		}
		return result;
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Generic rhythm game" />
</svelte:head>

<section>
	<h1>ASKL</h1>

	<audio bind:this={audioPlayer} src="/track-alpha9-nbts/song.mp3"></audio>

	<p>Type any of the A S K L, first hit starts the timer.</p>

	<div id="game-canvas" class="relative mx-auto w-[1024px] h-[700px] bg-[#aaa]">
		<!-- 4 hitpads -->
		{#each lanesGraphics as lane}
			<img
				src={imgHitpad}
				alt="hitpad"
				class="hdpi block absolute"
				style="left: {lane.end_xy[0] - graphicSettings.hitpadOffset[0]}px; top:{lane.end_xy[1] -
					graphicSettings.hitpadOffset[1]}px;"
			/>
		{/each}
	</div>

	<!-- Debugging text but can move intoconsole lah -->
	<p
		id="output-log"
		class="hidden container mx-auto w-[100vw] h-[800px] bg-black overflow-scroll font-mono text-white p-2"
	>
		{@html consoleOutputText}
	</p>

	<p>
		<!-- Display the parsed data -->
		{#each Object.entries(beatsStack) as [lane, times]}
			<div>
				<strong>{lane}:</strong>
				<ul>
					{#each times as time}
						<li>{time}</li>
					{/each}
				</ul>
			</div>
		{/each}
	</p>
</section>

<svelte:window on:keydown={onKeyDown} />
