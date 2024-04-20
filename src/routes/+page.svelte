<script>
	let keyA = 65;
	let keyS = 83;
	let keyK = 75;
	let keyL = 76;
	let keySpace = 32;

	let output_text = '';
	let wall_time_started_at = 0;
	let game_timer_paused_at = 0;
	let is_game_running = false;

	let audioPlayer;

	function onKeyDown(e) {
		if (![keyA, keyS, keyK, keyL, keySpace].includes(e.keyCode)) {
			return;
		}

		e.preventDefault();

		// Space bar starts-pauses the game
		if (e.keyCode == keySpace) {
			if (wall_time_started_at == 0 && game_timer_paused_at == 0) {
				// Start game
				wall_time_started_at = Date.now();
				audioPlayer.play();
				audioPlayer.fastSeek(0);
				is_game_running = true;
				output_text += `Game started at ${wall_time_started_at} <br>`;
			} else if (wall_time_started_at != 0 && game_timer_paused_at == 0) {
				// Pause game
				game_timer_paused_at = Date.now() - wall_time_started_at;
				audioPlayer.pause();
				is_game_running = false;
				output_text += `Game paused at ${game_timer_paused_at} <br>`;
			} else if (wall_time_started_at != 0 && game_timer_paused_at != 0) {
				// Resume from pause
				wall_time_started_at = wall_time_started_at + game_timer_paused_at;
				output_text += `Game resumed at ${game_timer_paused_at} <br>`;
				game_timer_paused_at = 0;
				audioPlayer.play();
				is_game_running = true;
			}
		}

		if ([keyA, keyS, keyK, keyL].includes(e.keyCode) && is_game_running) {
			let game_time = Date.now() - wall_time_started_at;
			output_text += `Key ${e.keyCode} hit at ${game_time} ms <br>`;
		}
	}

	function setupGame() {}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Generic rhythm game" />
</svelte:head>

<section>
	<h1>ASKL</h1>

	<audio bind:this={audioPlayer} src="/track-alpha9-nbts/song.mp3"></audio>

	<p>Type any of the A S K L, first hit starts the timer.</p>

	<p
		id="output-log"
		class="container mx-auto w-[800px] h-[200px] bg-black overflow-scroll font-mono text-white p-2"
	>
		{@html output_text}
	</p>
</section>

<svelte:window on:keydown={onKeyDown} />
