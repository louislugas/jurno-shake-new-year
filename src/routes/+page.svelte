<script>
	let sec = 30
	let tempX = 0, tempY = 0, tempZ = 0
	let distX = 0, distY = 0, distZ = 0
	let avgXYZ = 0, total = 0
	let countdown
	let startShake = false
	let step = 0

	function start() {
		startShake = true
		countdown = setInterval(() => {
			if (sec > 0) {
				sec--
			} else {
				clearInterval(countdown)
				step++
			}
			}, 1000)
	}

	

	// @ts-ignore
	function handleAcl(event) {
		distX = event.accelerationIncludingGravity.x - tempX
		distY = event.accelerationIncludingGravity.y - tempY
		distZ = event.accelerationIncludingGravity.z - tempZ
		
		avgXYZ = (Math.abs(distX)+Math.abs(distY)+Math.abs(distZ))/3
		
		if (avgXYZ > 1) {
			total = total + avgXYZ
		}
		
		tempX = event.accelerationIncludingGravity.x
		tempY = event.accelerationIncludingGravity.y
		tempZ = event.accelerationIncludingGravity.z
	}
</script>

<svelte:window on:devicemotion={sec > 0 && startShake ? handleAcl : null}></svelte:window>

<section>
	{#if step == 0}
		<div>
		<img src="./images/soda-can-01.svg" alt="kaleng">
		</div>

		<!-- <p>x: {distX.toFixed(2)}</p>
		<p>y: {distY.toFixed(2)}</p>
		<p>z: {distZ.toFixed(2)}</p> -->

		<!-- <h1>score:<br>{total.toFixed(2)}</h1> -->
		<!-- <div class="bar" style:width="{total}px"></div> -->

		<h1 id="timer">time:<br>{sec}</h1>
		{#if !startShake}
			<button on:click={start}>START</button>
		{/if}
	{:else if step == 1}
		<h1>your score:<br>{total.toFixed(2)}</h1>
	{/if}
</section>


<style>
	div {
		position: absolute;
	}
	.bar {
		width:0px;
		height:50px;
		background-color:red;
	}
	h1 {
		font-size:4rem;
		margin: 0;
		font-family: monospace;
		text-align: center;
	}
	button {
		font-size:3rem;
		margin-top: 2rem;
		cursor: pointer;
	}
	section {
		width:100%;
		height:100vh;
		display:flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}
	@media only screen and (max-width: 400px) {
		h1 {
			font-size: 2rem;
		}
		button {
			font-size: 1.5rem;
		}
	}
</style>