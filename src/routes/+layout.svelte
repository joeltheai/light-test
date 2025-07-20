<script lang="ts">
	let { children } = $props();
	import { ModeWatcher, toggleMode } from 'mode-watcher';
	import '../app.css';
</script>

<ModeWatcher />
<div class="fixed top-1 right-4 z-50">
	<button class="font-serif" onclick={toggleMode}>
		<div class="dark:hidden">L</div>
		<div class="hidden dark:block">L</div>
	</button>
	<button class="ml-2 font-serif" onclick={() => {
		let intervalId: number | null = (window as any).__modeToggleIntervalId || null;
		if (intervalId) {
			clearInterval(intervalId);
			(window as any).__modeToggleIntervalId = null;
		} else {
			intervalId = setInterval(() => {
				toggleMode();
			}, 100);
			(window as any).__modeToggleIntervalId = intervalId;
		}
	}}>
		<div>
			Cray
		</div>
	</button>
</div>
<div class="">
	{@render children()}
</div>

<style>
</style>
