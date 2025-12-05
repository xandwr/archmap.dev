<script lang="ts">
	interface Props {
		class?: string;
	}

	let { class: className = '' }: Props = $props();
</script>

<header
	class="border-b border-border bg-background/80 backdrop-blur-sm sticky top-0 z-50 {className}"
>
	<div class="mx-auto max-w-container-xl px-4 sm:px-6 lg:px-8">
		<nav class="flex h-16 items-center justify-between">
			<a
				href="/"
				class="text-xl font-semibold text-foreground hover:text-primary transition-colors"
			>
				archmap
			</a>

			<div class="flex items-center gap-6">
				<!-- Add nav links here as needed -->
				<!-- Example:
				<a href="/docs" class="text-sm text-muted-foreground hover:text-foreground transition-colors">
					Docs
				</a>
				-->
			</div>
		</nav>
	</div>
</header>
