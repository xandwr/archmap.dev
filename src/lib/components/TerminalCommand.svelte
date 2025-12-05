<script lang="ts">
	interface Props {
		command: string;
		class?: string;
	}

	let { command, class: className = '' }: Props = $props();
	let copied = $state(false);

	async function copyToClipboard() {
		await navigator.clipboard.writeText(command);
		copied = true;
		setTimeout(() => {
			copied = false;
		}, 2000);
	}
</script>

<div
	class="rounded-xl bg-muted shadow-lg overflow-hidden w-fit h-fit border border-neutral-500/50 {className}"
>
	<!-- Window chrome -->
	<div class="flex items-center justify-between px-4 py-3">
		<div class="flex items-center gap-2">
			<div class="w-3 h-3 rounded-full bg-[#ff5f57]"></div>
			<div class="w-3 h-3 rounded-full bg-[#febc2e]"></div>
			<div class="w-3 h-3 rounded-full bg-[#28c840]"></div>
		</div>
	</div>

	<!-- Command area -->
	<div class="flex items-center justify-between px-4 py-2 bg-background">
		<code class="font-mono text-foreground gap-x-3 flex flex-row">
			<span class="text-muted-foreground select-none">$</span>{command}
		</code>

		<button
			onclick={copyToClipboard}
			class="text-muted-foreground hover:text-foreground transition-colors cursor-pointer pl-4"
			aria-label={copied ? 'Copied!' : 'Copy to clipboard'}
		>
			{#if copied}
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="20"
					height="20"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<polyline points="20 6 9 17 4 12"></polyline>
				</svg>
			{:else}
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="20"
					height="20"
					viewBox="0 0 24 24"
					fill="none"
					stroke="currentColor"
					stroke-width="2"
					stroke-linecap="round"
					stroke-linejoin="round"
				>
					<rect width="14" height="14" x="8" y="8" rx="2" ry="2"></rect>
					<path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"></path>
				</svg>
			{/if}
		</button>
	</div>
</div>
