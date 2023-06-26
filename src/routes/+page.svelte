<script lang="ts">
	import { onMount } from 'svelte';

	let counter = 0;
	let referenceLength = 0;
	let stepValue = 0;
	let currentCurveLength = 0;
	let calibrated = false;

	const units = {
		inch: 'in',
		centimeter: 'cm'
	} as const;

	type Units = (typeof units)[keyof typeof units];

	let measureUnit: Units = units.centimeter;
	let history: String[] = [];

	onMount(() => {
		if (localStorage.getItem('stepValue')) {
			if (!Number.isNaN(Number.parseFloat(localStorage.getItem('stepValue')!))) {
				stepValue = Number.parseFloat(localStorage.getItem('stepValue')!);
			}
		}

		if (localStorage.getItem('history')) {
			history = JSON.parse(localStorage.getItem('history')!);
		}
	});

	const handleWheel = (e: WheelEvent) => {
		counter += Math.sign(e.deltaY);
		calculateLength();
	};

	const reset = (e: MouseEvent) => {
		if (e.altKey) {
			history = [...history, `${currentCurveLength} ${measureUnit}`];
			localStorage.setItem('history', JSON.stringify(history));
			counter = 0;
			currentCurveLength = 0;
			alert('Reset');
		}
	};

	const calibrate = () => {
		if (measureUnit === units.centimeter) {
			stepValue = referenceLength / counter;
		} else if (measureUnit === units.inch) {
			stepValue = (referenceLength * 2.54) / counter;
		}
		counter = 0;
		calibrated = true;
		if (!Number.isNaN(stepValue)) {
			localStorage.setItem('stepValue', stepValue.toString());
		}
	};

	const resetCaliber = () => {
		counter = 0;
		calibrated = false;
		referenceLength = 0;
		stepValue = 0;
		currentCurveLength = 0;
		localStorage.removeItem('stepValue');
	};

	const calculateLength = () => {
		if (measureUnit === units.centimeter) {
			currentCurveLength = counter * stepValue;
		} else if (measureUnit === units.inch) {
			currentCurveLength = (counter * stepValue) / 2.54;
		}
	};

	const clearHistory = () => {
		history = [];
		localStorage.removeItem('history');
	};
</script>

<svelte:window
	on:wheel|nonpassive|preventDefault={handleWheel}
	on:click|nonpassive|preventDefault={reset}
/>

<div>
	[SHOULD BE: {counter / 3.125 + ` cm`}]
</div>

<div>
	<p>
		1. Press Alt + LMB to reset to zero. <br />
		2.Input the length of the reference segment you use (e.g. a distance on a ruler, any segment you
		know the precise length of). <br />
		3.Then run the mouse wheel along it from start to end and press Calibrate. <br />
		All mouse manipulations must be done while the cursor is whithing the bounds of the app page.
	</p>
	<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<form class="box" on:submit|preventDefault={calibrate} on:click|stopPropagation>
		<input type="number" step=".01" bind:value={referenceLength} min="0" />
		<select bind:value={measureUnit} on:change={calculateLength}>
			{#each Object.values(units) as unit}
				<option value={unit}>
					{unit}
				</option>
			{/each}
		</select>
		<button type="submit">Calibrate</button>
		<p>Status: {calibrated ? 'Calibrated' : 'Not calibrated'}</p>
		<button type="button" on:click={resetCaliber}>Reset caliber</button>
	</form>
	<p>Wheel steps: {counter}</p>
	<p>Wheel step value: {stepValue} mm</p>
</div>

<div class="result">
	{#if measureUnit === units.centimeter}
		<p>Current curve length {currentCurveLength} cm</p>
	{/if}
	{#if measureUnit === units.inch}
		<p>Current curve length {currentCurveLength} in</p>
	{/if}
</div>

{#if history.length}
	<div class="history box">
		<strong>Recent results</strong>
		{#each history as entry, idx}
			<p>{idx + 1}: {entry}</p>
		{/each}
		<button on:click={clearHistory}>Clear history</button>
	</div>
{/if}

<style>
	.box {
		border: 1px solid #000;
		padding: 5px;
	}
</style>
