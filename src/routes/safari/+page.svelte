<script context="module">
	// @ts-nocheck
  import Device from 'svelte-device-info'
</script>

<script>
	import { onMount } from "svelte";
	import { browser } from '$app/environment';

	let tempX = 0, tempY = 0, tempZ = 0
	let distX = 0, distY = 0, distZ = 0
	let avgXYZ = 0, total = 0
	let startShake = false
	let mobile = false
	let ready = false
    let isSafari = false

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

	onMount(() => {
		mobile = Device.isMobile
		ready = true
        startShake = true
        if (browser) {
            let ua = window.navigator.userAgent.toLowerCase(); 

			if (ua.indexOf('safari') != -1) { 
				if (ua.indexOf('chrome') > -1) {
					isSafari = false
				} else {
					isSafari = true
				}
			}
        }
	})

</script>

<svelte:window on:devicemotion={startShake ? handleAcl : null}></svelte:window>
<svelte:head>
	<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
	<meta name="HandheldFriendly" content="true" />
</svelte:head>

<section>
{#if ready}
	{#if mobile}
		<h1>Safari Debugger</h1>
        <h3>Your browser is {isSafari ? 'Safari' : 'not Safari'}</h3>
        <br>
        <p>X: <strong>{distX.toFixed(2)}</strong></p>
        <p>Y: <strong>{distY.toFixed(2)}</strong></p>
        <p>Z: <strong>{distZ.toFixed(2)}</strong></p>
        <br>
        <p>Average: {avgXYZ.toFixed(2)}</p>
        <div 
            style:width="{avgXYZ*10}px"
            class="bar"></div>
	{:else}
		<h1>Play this game on your mobile phone</h1>
	{/if}
{/if}
</section>


<style>
	:global(body) {
		margin: 0;
		padding: 0;
		overflow: hidden;
		background-color: rgb(22, 18, 48);
	}
	h1 {
		font-size:4rem;
		margin: 0;
		font-family: monospace;
		text-align: center;
		color:whitesmoke;
	}
    h3 {
		font-size:1.5rem;
		margin: 0;
		font-family: monospace;
		text-align: center;
		color:whitesmoke;
	}
    p {
        margin: 0;
        font-size:1.2rem;
        color:whitesmoke;
        font-family: sans-serif;
    }
    .bar {
        margin-top: 2rem;
        background-color: orange;
        height:50px;
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
	}
</style>