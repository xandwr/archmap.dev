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

	// ANSI foreground color code to CSS class mapping
	const fgColors: Record<number, string> = {
		30: 'text-black',
		31: 'text-red-500',
		32: 'text-green-500',
		33: 'text-yellow-500',
		34: 'text-blue-500',
		35: 'text-purple-500',
		36: 'text-cyan-500',
		37: 'text-white',
		39: '', // default foreground
		90: 'text-gray-500',
		91: 'text-red-400',
		92: 'text-green-400',
		93: 'text-yellow-400',
		94: 'text-blue-400',
		95: 'text-purple-400',
		96: 'text-cyan-400',
		97: 'text-gray-100'
	};

	// ANSI background color code to CSS class mapping
	const bgColors: Record<number, string> = {
		40: 'bg-black',
		41: 'bg-red-500',
		42: 'bg-green-500',
		43: 'bg-yellow-500',
		44: 'bg-blue-500',
		45: 'bg-purple-500',
		46: 'bg-cyan-500',
		47: 'bg-white',
		49: '', // default background
		100: 'bg-gray-500',
		101: 'bg-red-400',
		102: 'bg-green-400',
		103: 'bg-yellow-400',
		104: 'bg-blue-400',
		105: 'bg-purple-400',
		106: 'bg-cyan-400',
		107: 'bg-gray-100'
	};

	interface AnsiState {
		bold: boolean;
		italic: boolean;
		underline: boolean;
		strikethrough: boolean;
		fgColor: string;
		bgColor: string;
	}

	function createDefaultState(): AnsiState {
		return {
			bold: false,
			italic: false,
			underline: false,
			strikethrough: false,
			fgColor: '',
			bgColor: ''
		};
	}

	function stateToClasses(state: AnsiState): string[] {
		const classes: string[] = [];
		if (state.bold) classes.push('font-bold');
		if (state.italic) classes.push('italic');
		if (state.underline) classes.push('underline');
		if (state.strikethrough) classes.push('line-through');
		if (state.fgColor) classes.push(state.fgColor);
		if (state.bgColor) classes.push(state.bgColor);
		return classes;
	}

	// Convert ANSI escape sequences to HTML with Tailwind classes
	// Returns array of HTML strings (one per line) with state carried across lines
	function ansiToHtmlLines(text: string): string[] {
		const lines: string[] = [];
		let currentLine = '';
		let state = createDefaultState();
		let spanOpen = false;
		let i = 0;

		// Normalize line endings
		text = text.replace(/\r\n/g, '\n').replace(/\r/g, '\n');

		while (i < text.length) {
			// Check for newline
			if (text[i] === '\n') {
				// Close span at end of line if open
				if (spanOpen) {
					currentLine += '</span>';
				}
				lines.push(currentLine);
				currentLine = '';
				// Reopen span at start of next line if we have active styles
				const classes = stateToClasses(state);
				if (classes.length > 0) {
					currentLine = `<span class="${classes.join(' ')}">`;
					spanOpen = true;
				} else {
					spanOpen = false;
				}
				i++;
				continue;
			}

			// Check for ANSI escape sequence
			if (text[i] === '\x1b' && text[i + 1] === '[') {
				// Find the end of the escape sequence
				const match = text.slice(i).match(/^\x1b\[([0-9;]*)([A-Za-z])/);
				if (match) {
					const params = match[1];
					const command = match[2];

					if (command === 'm') {
						// SGR (Select Graphic Rendition) - color/style codes
						const codes = params ? params.split(';').map(Number) : [0];

						for (let j = 0; j < codes.length; j++) {
							const code = codes[j];

							if (code === 0) {
								// Reset all
								if (spanOpen) {
									currentLine += '</span>';
									spanOpen = false;
								}
								state = createDefaultState();
							} else if (code === 1) {
								state.bold = true;
							} else if (code === 3) {
								state.italic = true;
							} else if (code === 4) {
								state.underline = true;
							} else if (code === 9) {
								state.strikethrough = true;
							} else if (code === 22) {
								state.bold = false;
							} else if (code === 23) {
								state.italic = false;
							} else if (code === 24) {
								state.underline = false;
							} else if (code === 29) {
								state.strikethrough = false;
							} else if (code in fgColors) {
								state.fgColor = fgColors[code];
							} else if (code in bgColors) {
								state.bgColor = bgColors[code];
							} else if (code === 38 && codes[j + 1] === 5) {
								// 256-color foreground: ESC[38;5;⟨n⟩m
								const colorNum = codes[j + 2];
								state.fgColor = `ansi-fg-${colorNum}`;
								j += 2;
							} else if (code === 48 && codes[j + 1] === 5) {
								// 256-color background: ESC[48;5;⟨n⟩m
								const colorNum = codes[j + 2];
								state.bgColor = `ansi-bg-${colorNum}`;
								j += 2;
							}
						}

						// Close existing span if open
						if (spanOpen) {
							currentLine += '</span>';
							spanOpen = false;
						}

						// Open new span if we have any styling
						const classes = stateToClasses(state);
						if (classes.length > 0) {
							currentLine += `<span class="${classes.join(' ')}">`;
							spanOpen = true;
						}
					}
					// Skip other escape sequences (cursor movement, clear, etc.)

					i += match[0].length;
					continue;
				}
			}

			// Escape HTML special characters
			if (text[i] === '<') {
				currentLine += '&lt;';
			} else if (text[i] === '>') {
				currentLine += '&gt;';
			} else if (text[i] === '&') {
				currentLine += '&amp;';
			} else {
				currentLine += text[i];
			}
			i++;
		}

		// Close any remaining span and add final line
		if (spanOpen) {
			currentLine += '</span>';
		}
		lines.push(currentLine);

		return lines;
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

					// Convert buffer to lines with ANSI styling preserved across lines
					lines = ansiToHtmlLines(buffer);

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
		<pre class="whitespace-pre-wrap wrap-break-word">{#each lines as line}{@html line +
					'\n'}{/each}</pre>
	</div>
</div>
