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

<div class="flex flex-col items-center h-[calc(100vh-4rem)] p-8">
	<div class="flex flex-col items-center gap-y-4">
		<pre class="text-center font-mono text-sm sm:text-base leading-none">{titleText}</pre>
		<TerminalCommand command="cargo install archmap" />
	</div>

	<TerminalOutput castUrl="/demos/demo.cast" class="flex-1 max-w-2xl w-full h-128 my-16" />
</div>
