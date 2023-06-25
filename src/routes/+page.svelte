<script lang="ts">
	let counter = 0;
  let referenceLength = 0
  let stepValue = 0
  let currentCurveLength = 0
    
  const units = {
    inch: 'inch',
    centimeter: 'centimeter'
  } as const

  type Units = typeof units[keyof typeof units]

  let measureUnit: Units = units.centimeter

	const handleWheel = (e: WheelEvent) => {
		counter += Math.sign(e.deltaY);
    calculateLength()
		console.log(counter);
	};

	const reset = (e: MouseEvent) => {
    if (e.altKey) counter = 0;
		console.log(counter);
	};

  const calibrate = () => {
    stepValue = referenceLength / counter
  }

  const calculateLength = () => {
    // value in mm:
    currentCurveLength = counter / stepValue
  }
</script>

<svelte:window
	on:wheel|nonpassive|preventDefault={handleWheel}
	on:click|nonpassive|preventDefault={reset}
/>

<div>
	SHOULD BE: {counter / 3.125 + ` cm`}
</div>

<div>
  <p>1. Press Alt + LMB to reset to zero. <br>
    2.Input the length of the reference segment you use (e.g. a distance on a ruler, any segment you know the precise length of). <br>
    3.Then run the mouse wheel along it from start to end and press Calibrate. <br> 
    All mouse manipulations must be done while the cursor is whithing the bounds of the app page.</p>
  <form on:submit|preventDefault={calibrate} on:click|stopPropagation>
    <input type="number" step=".01" bind:value={referenceLength} on:change={() => {console.log(referenceLength)}} min="0"/>
    <button type="submit">Calibrate</button>
  </form>
</div>

<select
bind:value={measureUnit}
>
  {#each Object.values(units) as unit}
    <option value={unit}>
      {unit}
    </option>
  {/each}
</select>

<p>stepValue {stepValue}</p>


<div>
  <p>Current curve length {currentCurveLength} mm</p>
</div>