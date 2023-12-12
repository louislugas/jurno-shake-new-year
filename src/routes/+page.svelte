<script context="module">
  import Device from 'svelte-device-info'
</script>

<script>
	import { onMount } from "svelte";
	import { browser } from '$app/environment';
	import { imgurl } from '$lib/imgurl';

	let isSafari = false

	let sec = 15, startSec = 4
	let tempX = 0, tempY = 0, tempZ = 0
	let distX = 0, distY = 0, distZ = 0
	let avgXYZ = 0, total = 0
	let countdown, scoreInterval
	let startShake = false
	let step = 0
	let endScore = 0
	let showScore = 0
	let width, height, y, by
	let showRestart = false
	let mobile = false
	let ready = false
	let fireWorks = false
	let safariMotion = false
	let tickAudio, tickAudio2
	let showStartSec = false
	let selector = 0

	let cx, cy, radius = 4, distance = 0

	function startIntro() {
		if (typeof DeviceMotionEvent.requestPermission === 'function') {
			DeviceMotionEvent.requestPermission()
				.then(permissionState => {
					if (permissionState === 'granted') {
						safariMotion = true
						step = 1
					}
				})
				.catch(console.error);
		} else {
			safariMotion = true
			step = 1
		// handle regular non iOS 13+ devices
		}
	}

	function degToRad(deg) {
		return deg / (180/Math.PI)
	}

	function timeCountDown() {
		countdown = setInterval(() => {
			if (sec > 0) {
				sec--
				if (sec <=3 && sec > 0) {
					tickAudio.play()
				} else if (sec == 0) {
					tickAudio2.play()
				}
			} else {
				clearInterval(countdown)
				step = 2
			}
		}, 1000)
	}

	function start() {
		showStartSec = true
		startSec--
		tickAudio.play()
		setInterval(() => {
			if (startSec > 1) {
				startSec--
				tickAudio.play()
			} else if (startSec == 1) {
				startSec--
				tickAudio2.play()
			} else if (startSec == 0) {
				startSec--
				startShake = true
			}
		},1000)

		setTimeout(() => {
			startShake = true
			timeCountDown()
		},4000)
		
	}

	$: if (step == 2) {
		startShake = false
		endScore = Math.round(total)
		if (endScore != 0) {
			setTimeout(() => {
				y=height/3
				cy=y
				scoreInterval = setInterval(() => {
					if (showScore < endScore) {
						showScore++
						by++
					} else {
						clearInterval(scoreInterval)
						fireWorks = true
						distance = width/3
						setTimeout(() => {
							showRestart = true
						},1000)
					}
				},200)
			},500)
		}
	}

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
		if (browser) {
			let ua = window.navigator.userAgent.toLowerCase(); 

			if (ua.indexOf('safari') != -1) { 
				if (ua.indexOf('chrome') > -1) {
					isSafari = false
				} else {
					isSafari = false //temp
				}
			}
		}

		y=height-(width/25*2)
		by=height/2
		mobile = Device.isMobile
		tickAudio = new Audio("./audio/timer-01.mp3")
		tickAudio2 = new Audio("./audio/timer-02.mp3")
		ready = true
		selector = Math.round(Math.random()*3)
	})

	// function startShowScore() {
	// 	startShake = false
	// 	endScore = Math.round(total)
	// 	if (endScore != 0) {
	// 		setTimeout(() => {
	// 			y=height/3
	// 			cy=y
	// 			scoreInterval = setInterval(() => {
	// 				if (showScore < endScore) {
	// 					showScore++
	// 					by++
	// 				} else {
	// 					clearInterval(scoreInterval)
	// 					distance = 0
	// 					setTimeout(() => {
	// 						distance = width/3
	// 					},200)
	// 					setTimeout(() => {
	// 						showRestart = true
	// 					},1000)
	// 				}
	// 			},200)
	// 		},500)
	// 	}
	// }

	function restart() {
		location.reload()
	}
</script>

<svelte:window on:devicemotion={sec > 0 && startShake && safariMotion && step == 1 ? handleAcl : null}></svelte:window>
<svelte:head>
	<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
	<meta name="HandheldFriendly" content="true" />
	{#each imgurl as url}
		<link rel="preload" href={url} as="image" />
	{/each}
</svelte:head>

<section bind:clientWidth={width} bind:clientHeight={height}>
{#if ready}
	{#if mobile}
		{#if step == 0}
			<div 
				style:flex-direction="column"
				style:z-index="3"
			>
				<h3>Instruction:</h3>
				<h3>Shake your phone!</h3>
				<img src="./images/shake-frame.gif" alt="instruction">
				<button on:click={startIntro}>PLAY</button>
			</div>
		{:else if step == 1}
			<div>
				<img 
					style:transform={avgXYZ > 1 ? `translate(${avgXYZ*5 * (Math.round(Math.random()) == 1 ? 1 : -1 )}px,${avgXYZ*5 * (Math.round(Math.random()) == 1 ? 1 : -1 )}px)rotate(${avgXYZ*5 * (Math.round(Math.random()) == 1 ? 1 : -1 )}deg)
					` : `translate(0,0) rotate(0deg)` }
					src={imgurl[selector]} alt="kaleng">
			</div>
			<div style:z-index="10">
				{#if startSec == 0}
				<h1 style:z-index="10" class="warning">SHAKE YOUR PHONE</h1>
				{/if}
			</div>
			{#if startShake}
			<h1 class="timer">time:<br>{sec}</h1>
			{/if}
			{#if !startShake}
				{#if showStartSec}
					<h1 class="timer">{startSec}</h1>
				{:else}
					<button on:click={start}>START</button>
				{/if}
				
			{/if}
		{:else if step == 2}
			<div class="gradient"></div>
			<svg width="100%" height="100%">
				<g transform="translate(0 {by/100})">
					{#each Array(15) as _,i}
						<circle 
							class="star"
							cx={Math.random()*width} 
							cy={Math.random()*height}
							r={2}
							fill="white"></circle>
					{/each}
				</g>
					
				<path
					d="
						M0,{by} 
						l {width/5},0
						l 0,{height/6}
						l {width/5},0
						l 0,{-height/10}
						l {width/5},0
						l 0,{height/12}
						l {width/5},0
						l 0,{-height/8}
						l {width/5},0
						l 0,{height}
						l {-width},0"
					fill="#454159"
				></path>

				{#if !fireWorks}
					<rect 
						style:transition-duration="{endScore/2}ms"
						width={width/25} 
						height={width/25*2} 
						x={width/2-(width/25/2)} 
						y={y}
						fill="{
							selector == 0 ? "#e74b4c" :
							selector == 1 ? "#51aade" : 
							selector == 2 ? "#72be44" :
							selector == 3 ? "#e98a4a" :
							"#e74b4c"

						}"></rect>
				{/if}
				<rect
					style:transition-duration="{endScore/2}ms"
					width={width/40} 
					height={height} 
					x={width/2-(width/40/2)} 
					y={y+(width/25*2)}
					fill="#f4bd40">
				</rect>
				{#each Array(8) as _, i}
					<circle 
						class="spark"
						opacity={fireWorks ? 1 : 0}
						cx={(width/2)+Math.sin(degToRad(360/8*i))*distance} 
						cy={cy+Math.cos(degToRad(360/8*i))*distance} 
						r={radius}
						fill="white">
					</circle>
				{/each}
				
			</svg>
			
			<h1>your score:<br>{showScore.toLocaleString("de-DE")}</h1>
			{#if showRestart}
				<button on:click={restart}>RESTART</button>
			{/if}
		{/if}
	{:else}
		<h1>Play this game on your mobile phone</h1>
	{/if}
<!-- {:else}
	<h1>Loading...</h1> -->
{/if}
</section>


<style>
	:global(body) {
		margin: 0;
		padding: 0;
		overflow: hidden;
		background-color: rgb(22, 18, 48);
	}
	svg {
		position: absolute;
		border: 1px solid;
		z-index:2;
	}
	.gradient {
		width:100%;
		height:100%;
		background: linear-gradient(#171331, #242155);
		position: absolute;
		z-index:0;
	}
	path {
		transition: all 200ms linear;
	}
	rect {
		transition:all cubic-bezier(0.25, 1, 0.5, 1);
	}
	.spark {
		transition:all 500ms cubic-bezier(0.25, 1, 0.5, 1);
	}
	div {
		position: absolute;
		width:100%;
		display: flex;
		justify-content: center;
		align-items: center;
		z-index:-1;
	}
	img {
		width:50%;
		max-width:250px;
	}
	.timer {
		color:black;
	}
	.warning {
		color:black;
		background-color:red;
	}
	h3 {
		font-size:2rem;
		margin: 0;
		font-family: monospace;
		text-align: center;
		color:whitesmoke;
	}
	h1 {
		font-size:4rem;
		margin: 0;
		font-family: monospace;
		text-align: center;
		color:whitesmoke;
		z-index:5;
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