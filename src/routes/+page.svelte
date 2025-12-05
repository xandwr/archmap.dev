<script>
	import { onMount } from 'svelte';
	import 'asciinema-player/dist/bundle/asciinema-player.css';

	let container;

	onMount(async () => {
		const AsciinemaPlayer = await import('asciinema-player');
		AsciinemaPlayer.create('/demos/demo.cast', container);
	});
</script>

<div bind:this={container}></div>
