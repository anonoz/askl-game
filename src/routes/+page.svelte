<script>
	let keyA = 65;
	let keyS = 83;
	let keyK = 75;
	let keyL = 76;

	let output_text = '';
	let timer = 0;

	function onKeyDown(e) {
		output_text += `Key ${e.keyCode} hit at ${timer} <br>`;
	}
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Generic rhythm game" />
</svelte:head>

<section>
	<h1>ASKL</h1>

	<p>Type any of the A S K L, first hit starts the timer.</p>

	<p
		id="output-log"
		class="container mx-auto w-[800px] h-[200px] bg-black overflow-scroll font-mono text-white p-2"
	>
		{@html output_text}
	</p>
</section>

<svelte:window on:keydown|preventDefault={onKeyDown} />
