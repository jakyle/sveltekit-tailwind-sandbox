<script lang="ts">
	import DragAndDrop from '$lib/DragAndDrop.svelte';
	import NumberSlider from '$lib/NumberSlider.svelte';
	import ToolTipTest from '$lib/ToolTipTest.svelte';

	let min = 1;
	let max = 12;
</script>

<main class="flex h-screen w-screen items-center justify-center bg-stone-950">
	<DragAndDrop />
</main>