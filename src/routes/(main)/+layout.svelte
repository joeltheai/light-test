<script lang="ts">
	let { children } = $props();
	import { ModeWatcher, toggleMode } from 'mode-watcher';

	let isCrayMode = $state(false);

	function toggleCrayMode() {
		isCrayMode = !isCrayMode;
		if (isCrayMode) {
			document.documentElement.classList.add('cray');
		} else {
			document.documentElement.classList.remove('cray');
		}
	}
</script>

<ModeWatcher />
<div class="fixed top-1 right-4 z-50">
	<button class="font-serif" onclick={toggleMode}>
		<div class="dark:hidden">L</div>
		<div class="hidden dark:block">L</div>
	</button>
	<button class="ml-2 font-serif" onclick={toggleCrayMode}>
		<div class={isCrayMode ? 'text-primary' : ''}>
			Cray
		</div>
	</button>
</div>
<div class="">
	{@render children()}
</div>

<style>
</style>
