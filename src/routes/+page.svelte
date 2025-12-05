<script lang="ts">
	import figlet from 'figlet';
	import { onMount } from 'svelte';
	import TerminalCommand from '$lib/components/TerminalCommand.svelte';
	import TerminalOutput from '$lib/components/TerminalOutput.svelte';

	let titleText = $state('');

	onMount(async () => {
		figlet.text('archmap', { font: 'Nancyj-Underlined' }, (err, result) => {
			if (result) titleText = result;
		});
	});
</script>

<div class="flex flex-col items-center justify-center gap-y-4 p-8">
	<pre class="text-center font-mono text-sm sm:text-base leading-none">{titleText}</pre>
	<TerminalCommand command="cargo install archmap" />
	<TerminalOutput castUrl="/demos/demo.cast" class="max-w-2xl w-full h-[500px]" />
</div>
