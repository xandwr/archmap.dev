<script lang="ts">
	import { onMount } from 'svelte';
	import 'asciinema-player/dist/bundle/asciinema-player.css';

	let container: HTMLDivElement;

	onMount(async () => {
		const AsciinemaPlayer = await import('asciinema-player');
		AsciinemaPlayer.create('/demos/demo.cast', container);
	});
</script>

<div class="flex flex-1 items-center justify-center py-12 px-4">
	<div class="max-w-2xl w-full" bind:this={container}></div>
</div>
