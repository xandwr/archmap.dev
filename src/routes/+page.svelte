<script lang="ts">
	import { onMount } from 'svelte';
	import 'asciinema-player/dist/bundle/asciinema-player.css';

	let container: HTMLDivElement;

	onMount(async () => {
		const AsciinemaPlayer = await import('asciinema-player');
		AsciinemaPlayer.create('/demos/demo.cast', container);
	});
</script>

<main class="min-h-screen flex items-center justify-center">
	<div class="max-w-2xl w-full" bind:this={container}></div>
</main>
