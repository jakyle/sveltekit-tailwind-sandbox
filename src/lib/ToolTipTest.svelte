<script lang="ts">
	let popOverElement: null | HTMLDivElement = null;
	let timeoutTime = 600;

	let timeoutId: number | null = null;
	const handleMouseEnter = () => {
		timeoutId = setTimeout(() => popOverElement?.showPopover(), timeoutTime);
	};

	const handleMouseLeave = () => {
		if (timeoutId) {
			clearTimeout(timeoutId);
			timeoutId = null;
		}

		popOverElement?.hidePopover();
	};
</script>

<div
	class="relative flex size-36 items-center justify-center overflow-hidden rounded bg-neutral-400"
>
	<button
		on:mouseenter|preventDefault={handleMouseEnter}
		on:mouseleave|preventDefault={handleMouseLeave}
		class="rounded border border-black bg-white px-2 py-1"
		popovertarget="mypopover"
	>
		Toggle
	</button>

	<div
		on:beforetoggle={() => console.log('beforetoggle')}
		class="top-16 px-1 py-0.5 text-sm"
		id="mypopover"
		popover="manual"
		bind:this={popOverElement}
	>
		Popover content
	</div>
</div>
