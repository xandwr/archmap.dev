<script lang="ts">
	import { onMount } from 'svelte';

	interface Props {
		castUrl: string;
		autoPlay?: boolean;
		speed?: number;
		class?: string;
	}

	let { castUrl, autoPlay = true, speed = 1, class: className = '' }: Props = $props();

	let lines = $state<string[]>([]);
	let container: HTMLDivElement;
	let playing = $state(false);

	// ANSI color code to CSS class mapping
	const ansiColors: Record<number, string> = {
		30: 'text-black',
		31: 'text-red-500',
		32: 'text-green-500',
		33: 'text-yellow-500',
		34: 'text-blue-500',
		35: 'text-purple-500',
		36: 'text-cyan-500',
		37: 'text-white',
		90: 'text-gray-500',
		91: 'text-red-400',
		92: 'text-green-400',
		93: 'text-yellow-400',
		94: 'text-blue-400',
		95: 'text-purple-400',
		96: 'text-cyan-400',
		97: 'text-gray-100'
	};

	// Convert ANSI escape sequences to HTML with Tailwind classes
	function ansiToHtml(text: string): string {
		let result = '';
		let currentClasses: string[] = [];
		let i = 0;

		while (i < text.length) {
			// Check for ANSI escape sequence
			if (text[i] === '\x1b' && text[i + 1] === '[') {
				const endIndex = text.indexOf('m', i);
				if (endIndex !== -1) {
					const codes = text
						.slice(i + 2, endIndex)
						.split(';')
						.map(Number);

					for (const code of codes) {
						if (code === 0) {
							// Reset
							if (currentClasses.length > 0) {
								result += '</span>';
								currentClasses = [];
							}
						} else if (code === 1) {
							// Bold
							if (currentClasses.length > 0) result += '</span>';
							currentClasses.push('font-bold');
							result += `<span class="${currentClasses.join(' ')}">`;
						} else if (code === 4) {
							// Underline
							if (currentClasses.length > 0) result += '</span>';
							currentClasses.push('underline');
							result += `<span class="${currentClasses.join(' ')}">`;
						} else if (ansiColors[code]) {
							if (currentClasses.length > 0) result += '</span>';
							currentClasses = [ansiColors[code]];
							result += `<span class="${currentClasses.join(' ')}">`;
						}
					}

					i = endIndex + 1;
					continue;
				}
			}

			// Check for other escape sequences to skip (cursor movement, clear screen, etc.)
			if (text[i] === '\x1b' && text[i + 1] === '[') {
				const match = text.slice(i).match(/^\x1b\[[0-9;]*[A-Za-z]/);
				if (match) {
					i += match[0].length;
					continue;
				}
			}

			// Escape HTML special characters
			if (text[i] === '<') {
				result += '&lt;';
			} else if (text[i] === '>') {
				result += '&gt;';
			} else if (text[i] === '&') {
				result += '&amp;';
			} else {
				result += text[i];
			}
			i++;
		}

		if (currentClasses.length > 0) {
			result += '</span>';
		}

		return result;
	}

	// Process output and split into lines
	function processOutput(output: string): string[] {
		// Handle carriage returns and newlines
		const processed = output.replace(/\r\n/g, '\n').replace(/\r/g, '\n');

		return processed.split('\n');
	}

	async function playCast() {
		if (playing) return;
		playing = true;
		lines = [];

		try {
			const response = await fetch(castUrl);
			const text = await response.text();
			const castLines = text.trim().split('\n');

			// Skip header (first line)
			const events = castLines.slice(1).map((line) => JSON.parse(line));

			let buffer = '';
			let lastTime = 0;

			for (const event of events) {
				const [time, type, data] = event;

				if (type === 'o') {
					// Wait for the appropriate time
					const delay = ((time - lastTime) * 1000) / speed;
					if (delay > 0) {
						await new Promise((resolve) => setTimeout(resolve, delay));
					}
					lastTime = time;

					// Append to buffer and process
					buffer += data;

					// Convert buffer to lines
					const newLines = processOutput(buffer);

					// Update displayed lines
					lines = newLines.map((line) => ansiToHtml(line));

					// Auto-scroll to bottom
					if (container) {
						requestAnimationFrame(() => {
							container.scrollTop = container.scrollHeight;
						});
					}
				}
			}
		} catch (error) {
			console.error('Failed to load cast file:', error);
		}

		playing = false;
	}

	function restart() {
		lines = [];
		playCast();
	}

	onMount(() => {
		if (autoPlay) {
			playCast();
		}
	});
</script>

<div
	class="flex flex-col bg-black rounded-xl border-2 border-neutral-500/50 overflow-hidden {className}"
>
	<!-- Terminal chrome -->
	<div
		class="flex items-center justify-between px-4 py-3 bg-neutral-800 border-b border-neutral-700"
	>
		<div class="flex items-center gap-2">
			<div class="w-3 h-3 rounded-full bg-[#ff5f57]"></div>
			<div class="w-3 h-3 rounded-full bg-[#febc2e]"></div>
			<div class="w-3 h-3 rounded-full bg-[#28c840]"></div>
		</div>
		<button
			onclick={restart}
			class="text-neutral-400 hover:text-white transition-colors text-sm"
			aria-label="Restart demo"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="16"
				height="16"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
				<path d="M3 3v5h5" />
			</svg>
		</button>
	</div>

	<!-- Terminal content - scrollable -->
	<div
		bind:this={container}
		class="flex-1 overflow-y-auto overflow-x-hidden p-4 font-mono text-sm text-green-400 leading-relaxed"
	>
		<pre class="whitespace-pre-wrap break-words">{#each lines as line}{@html line}
			{/each}</pre>
	</div>
</div>
