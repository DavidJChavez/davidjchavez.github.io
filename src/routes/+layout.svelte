<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<div class="grid min-h-dvh grid-cols-[5rem_1fr_5rem] grid-rows-[5rem_auto_5rem]">
	<div class="border-r border-b border-gray-300"></div>
	<div class="border-b border-gray-300"></div>
	<div class="border-b border-l border-gray-300"></div>
	<div class="border-r border-gray-300"></div>
	<main>
		{@render children()}
	</main>
	<div class="border-l border-gray-300"></div>
	<div class="border-t border-r border-gray-300"></div>
	<div class="border-t border-gray-300"></div>
	<div class="border-t border-l border-gray-300"></div>
</div>
