<script lang="ts">
	import { createEventDispatcher, onMount } from 'svelte';

	export let min = 1;
	export let max = 10;
	export let step = 1;
	export let value = min;

	if (min > max) {
		throw new Error('min must be less than or equal to max');
	}

	const dispatch = createEventDispatcher<{ change: number }>();
	$: values = Array.from({ length: Math.floor((max - min) / step) + 1 }, (_, i) => min + i * step);
	$: firstValue = values.at(0) ?? min;
	$: lastValue = values.at(-1) ?? max;

	let track: HTMLDivElement;
	let slider: HTMLButtonElement;
	let elements: Record<number, HTMLButtonElement> = {};

	onMount(() => {
		setTrackerLeft(value);
	});

	let isSliding = false;
	const handleValueSelect = (val: number) => {
		value = val;
		dispatch('change', val);
	};

	const handleSlideValueClick = (event: MouseEvent) => {
		const val = parseInt((event.target as HTMLButtonElement).dataset.value ?? '1');
		handleSlideValueSelect(val);
	};

	const handleSlideValueSelect = (val: number) => {
		handleValueSelect(val);
		setTrackerLeft(val);
	};

	let trackerLeft = 0;
	let leftOffset = 0;

	const handleTrackerMouseDown = (event: MouseEvent) => {
		isSliding = true;
		leftOffset = event.clientX - trackerLeft;
	};

	const handleTrackerMouseUp = () => {
		if (isSliding) {
			isSliding = false;
			setTrackerLeft(value);
		}
	};

	const handleMouseMove = (event: MouseEvent) => {
		if (isSliding) {
			const newLeft = event.clientX - leftOffset;
			const left = Math.max(0, Math.min(newLeft, track.clientWidth - slider.offsetWidth));
			trackerLeft = left;

			const { width } = track.getBoundingClientRect?.() ?? { width: 0 };
			const lengthOfEachValueHalfwayPoint = width / values.length / 3;
			const idx = Math.ceil(((left + lengthOfEachValueHalfwayPoint) / width) * values.length) - 1;
			const val = values[idx];

			if (value !== val) {
				handleValueSelect(val);
			}
		}
	};

	const handleKeyDown = (event: KeyboardEvent) => {
		let newVal = value;
		if (event.key === 'ArrowLeft' || event.key === 'ArrowDown') {
			newVal = Math.max(firstValue, value - step);
		} else if (event.key === 'ArrowRight' || event.key === 'ArrowUp') {
			newVal = Math.min(lastValue, value + step);
		} else if (event.key === 'End') {
			newVal = lastValue;
		} else if (event.key === 'Home') {
			newVal = firstValue;
		} else if (event.key === 'PageUp') {
			newVal = Math.min(lastValue, value + step * 2);
		} else if (event.key === 'PageDown') {
			newVal = Math.max(firstValue, value - step * 2);
		}

		if (newVal !== value) {
			handleSlideValueSelect(newVal);
		}
	};

	const setTrackerLeft = (val: number) => {
		const elementLeft = elements[val].getBoundingClientRect?.() ?? { left: 0 };
		const trackRect = track.getBoundingClientRect?.() ?? { left: 0 };
		trackerLeft = elementLeft.left - trackRect.left - 0.5;
	};

	const getButtonClasses = (val: number, value: number, isSliding: boolean) => {
		const baseClasses =
			'z-20 flex size-6 items-center justify-center bg-transparent pb-0.5 font-semibold'.split(' ');

		if (val === firstValue) {
			baseClasses.push('rounded-l-2xl');
		}

		if (val === lastValue) {
			baseClasses.push('rounded-r-2xl');
		}

		if (val < value) {
			baseClasses.push('active:bg-sky-400/40 hover:bg-sky-300/40 text-white');
		}

		if (val > value) {
			baseClasses.push('active:bg-sky-200 hover:bg-sky-100 text-black');
		}

		if (isSliding || val === value) {
			baseClasses.push('pointer-events-none');
		} else {
			baseClasses.push('pointer-events-auto');
		}

		if (!isSliding) {
			baseClasses.push('transition-all');
		}

		return baseClasses.join(' ');
	};
</script>

<svelte:document on:mouseup={handleTrackerMouseUp} on:mousemove={handleMouseMove} />

<div
	bind:this={track}
	class="relative isolate flex items-center divide-x divide-stone-400/60 rounded-2xl border border-black bg-white"
>
	<button
		bind:this={slider}
		on:mousedown={handleTrackerMouseDown}
		on:keydown={handleKeyDown}
		role="slider"
		aria-valuenow={value}
		aria-valuemin={firstValue}
		aria-valuemax={lastValue}
		style:left={`${trackerLeft}px`}
		class:transition-all={!isSliding}
		class:scale-125={isSliding}
		class="absolute z-30 flex size-6 cursor-grab items-center justify-center rounded-full border border-white bg-black pb-0.5 font-bold text-white outline-2 outline-sky-400 hover:scale-125 focus:scale-125 focus:outline"
	>
		{value}
	</button>
	<div
		class:transition-all={!isSliding}
		style:width={`${trackerLeft + slider?.clientWidth ?? 24}px`}
		class="pointer-events-none absolute z-10 h-full rounded-2xl bg-purple-600"
	></div>
	{#each values as val}
		<button
			bind:this={elements[val]}
			on:click={handleSlideValueClick}
			data-value={val}
			class={getButtonClasses(val, value, isSliding)}
		>
			{val}
		</button>
	{/each}
</div>
