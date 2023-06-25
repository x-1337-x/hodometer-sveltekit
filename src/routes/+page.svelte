<script lang="ts">
	let counter = 0;

	const handleWheel = (e: WheelEvent) => {
		counter += e.deltaY;
		console.log(counter);
	};

	const reset = (e: MouseEvent) => {
		counter = 0;
		console.log(counter);
	};
</script>

<svelte:window
	on:wheel|nonpassive|preventDefault={handleWheel}
	on:click|nonpassive|preventDefault={reset}
/>

<div>
	Curve length: {counter / 300 / 3.125 + ` cm`}
</div>
