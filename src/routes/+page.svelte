<script lang="ts">
	// Libs
	import { onMount, tick } from 'svelte';
	import { writable } from 'svelte/store';
	import { fade, fly } from 'svelte/transition';

	// Image assets
	import imgHitpad from '$lib/images/hitpad@3x.png';
	import imgBallLane1 from '$lib/images/ball-lane-1@3x.png';
	import imgBallLane2 from '$lib/images/ball-lane-2@3x.png';
	import imgBallLane3 from '$lib/images/ball-lane-3@3x.png';
	import imgBallLane4 from '$lib/images/ball-lane-4@3x.png';
	import Counter from './Counter.svelte';
	import { cubicIn } from 'svelte/easing';

	const keyA = 65;
	const keyS = 83;
	const keyK = 75;
	const keyL = 76;
	const keySpace = 32;
	let chartCsv: String;

	const graphicSettings = {
		beatDuration: 3, // seconds
		hitpadOffset: [116 / 2, 62 / 2],
		laneBallOffset: 0
	};
	const lanesGraphics = [
		{
			ball_graphic: imgBallLane1,
			start_xy: [100, 100],
			end_xy: [100, 600]
		},
		{
			ball_graphic: imgBallLane2,
			start_xy: [300, 100],
			end_xy: [300, 600]
		},
		{
			ball_graphic: imgBallLane3,
			start_xy: [500, 100],
			end_xy: [500, 600]
		},
		{
			ball_graphic: imgBallLane4,
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
	let beatsStack: Array<Array<Number>> = {};
	let ballsInLane = [[], [], [], []];

	let audioPlayer: HTMLMediaElement;

	onMount(async () => {
		// Load the song, chart, bind
		const chartCsvResponse = await fetch('/track-alpha9-nbts/chart.csv');
		chartCsv = await chartCsvResponse.text();
	});

	// This doesn't start the game
	function setupGame() {
		console.log('Running setupGame();');
		// Load the beats stack
		beatsStack = parseChartCSV(chartCsv);
		console.log(beatsStack);

		// Reset game states
		currentScore = 0;
		audioPlayer.fastSeek(0);
		audioPlayer.play();

		// // Bind rendering event
		for (let lane_id in beatsStack) {
			for (let timing of beatsStack[lane_id]) {
				setTimeout(
					() => {
						ballsInLane[lane_id] = ballsInLane[lane_id] + [timing * 1000];
						console.log(`Adding ball ${lane_id} for time ${timing * 1000}`);

						setTimeout(() => {
							console.log(`Removing ball ${lane_id} #${timing * 1000}`);
							ballsInLane[lane_id] = ballsInLane[lane_id].slice(1);
						}, 500);
					},
					1000.0 * (timing - graphicSettings.beatDuration)
				);
			}
		}
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
				setupGame();
				return;
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
	function parseChartCSV(csv: String) {
		let lines = csv.split('\n');
		let result: Array<Array<Number>> = [];
		// Skip header line, iterate from second line
		for (let i = 1; i < lines.length; i++) {
			if (!lines[i]) continue;
			const [strikeAt, lane] = lines[i].split(',');
			const lane_id = parseInt(lane.trim()) - 1;
			if (!result[lane_id]) {
				result[lane_id] = [];
			}
			result[lane_id].push(parseFloat(strikeAt.trim()));
		}
		return result;
	}
</script>

<svelte:window on:keydown={onKeyDown} />

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

		{#each ballsInLane as laneBalls, laneId}
			{#each laneBalls as balls}
				<img
					in:fly={{ y: -1400, duration: 1200 }}
					out:fade
					src={lanesGraphics[laneId].ball_graphic}
					alt="hitpad"
					class="hdpi block absolute"
					style="left: {lanesGraphics[laneId].end_xy[0] -
						graphicSettings.laneBallOffset}px; top:{lanesGraphics[laneId].end_xy[1] -
						graphicSettings.laneBallOffset}px;"
				/>
			{/each}
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
