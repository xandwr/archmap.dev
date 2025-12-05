<script lang="ts">
	import figlet from 'figlet';
	import { onMount } from 'svelte';
	import 'asciinema-player/dist/bundle/asciinema-player.css';
	import TerminalCommand from '$lib/components/TerminalCommand.svelte';

	let container: HTMLDivElement;
	let titleText = $state('');

	onMount(async () => {
		figlet.text('archmap', { font: 'Nancyj-Underlined' }, (err, result) => {
			if (result) titleText = result;
		});

		const AsciinemaPlayer = await import('asciinema-player');
		AsciinemaPlayer.create('/demos/demo.cast', container, {
			autoPlay: true,
			controls: false,
			loop: false,
			fit: 'height',
			terminalFontFamily: 'monospace'
		});
	});
</script>

<div class="flex flex-col items-center justify-center gap-y-4 p-8">
	<pre class="text-center font-mono text-sm sm:text-base leading-none">{titleText}</pre>
	<TerminalCommand command="cargo install archmap" />
	<div
		class="max-w-2xl w-full h-[500px] overflow-y-auto border-2 border-neutral-500/50 rounded-xl bg-black"
		bind:this={container}
	></div>
</div>
