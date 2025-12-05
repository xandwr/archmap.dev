<script lang="ts">
	interface Props {
		class?: string;
	}

	let { class: className = '' }: Props = $props();
</script>

<footer class="border-t border-border bg-muted/50 {className}">
	<div class="mx-auto max-w-container-xl px-4 sm:px-6 lg:px-8">
		<div class="flex h-16 items-center justify-between">
			<p class="text-sm text-muted-foreground">archmap</p>

			<div class="flex items-center gap-6">
				<!-- Add footer links here as needed -->
				<!-- Example:
				<a href="/privacy" class="text-sm text-muted-foreground hover:text-foreground transition-colors">
					Privacy
				</a>
				-->
			</div>
		</div>
	</div>
</footer>
