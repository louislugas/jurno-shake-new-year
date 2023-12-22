<script context="module">
    import Device from 'svelte-device-info'
</script>

<script>
    import { onMount } from "svelte";
    import { browser } from '$app/environment';
    import { imgurl } from '$lib/imgurl';
    import supabase from '$lib/supabase'

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
    let num = 200
	let showLeaderboard = false
	let leaderBoard, leaderBoardDialog
	let scoreArray, inputName, submitdisable = false
	let shortLeaderBoard

    let cx, cy, radius = 2, distance = 0

    async function getData() {
		let { data, error } = await supabase
			.from('shake_score')
			.select('name, score')
		leaderBoard = data

		if(error) {
			window.alert(error.message+"\nPlease check your connection and refresh your page")
		}
    }

	async function insertData() {
		if (inputName != null || inputName != undefined) {
            if (Math.round(showScore) > Math.min(...scoreArray)) {

                    let {data, error} = await supabase
                        .from("shake_score")
                        .insert({name: inputName, score: showScore})
                        .select()
                    
                    if (data) {
                        shortLeaderBoard.push({
                            name: data[0].name,
                            score: data[0].score
                            })
						shortLeaderBoard = shortLeaderBoard
                        shortLeaderBoard = shortLeaderBoard.sort((a,b) => b.score - a.score)
						shortLeaderBoard = shortLeaderBoard

                        submitdisable = true
                        window.alert("Score successfully submitted")
                    }
                        
                    if (error) {
                        window.alert(error.message)
                    }
            } 
		} else {
			window.alert("Please insert name")
		}
		
	}

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
                        distance = width/2.5
                        
                        setTimeout(() => {
                            showLeaderboard = true
							showRestart = true
							leaderBoardDialog.showModal()
                        },500)
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

    onMount(async() => {
        await getData()

		shortLeaderBoard = leaderBoard.sort((a,b) => b.score - a.score).slice(0,10)
		shortLeaderBoard = shortLeaderBoard
		scoreArray = shortLeaderBoard.map(d => d.score)

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
        // selector = 5
    })

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
    <link rel="preconnect" href="https://fonts.googleapis.com">
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
	<link href="https://fonts.googleapis.com/css2?family=Luckiest+Guy&display=swap" rel="stylesheet">
</svelte:head>

<section bind:clientWidth={width} bind:clientHeight={height}>
{#if ready}
    {#if mobile}
        {#if step == 0}
            <div 
                style:flex-direction="column"
                style:z-index="3"
            >
                <h3 class="text-shadow">Instruction:</h3>
                <h3 class="text-shadow">Shake your phone!</h3>
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
                <h1 style:z-index="10" class="warning text-shadow">SHAKE YOUR PHONE</h1>
                {/if}
            </div>
            {#if startShake}
            <h1 class="timer text-shadow">
                <!-- time:<br> -->
                {sec}</h1>
            {/if}
            {#if !startShake}
                {#if showStartSec}
                    <h1 class="timer text-shadow">{startSec}</h1>
                {:else}
                    <button on:click={start}>START</button>
                {/if}
                
            {/if}
        {:else if step == 2}
            <div class="gradient"></div>
            <svg width="100%" height="100%">
                <g transform="translate(0 {by/100})">
                    {#each Array(15) as _,i}
                        <image
                            class="star"
                            x={Math.random()*width-5}
                            y={Math.random()*height-5}
                            width={8}
                            height={8}
                            href="./images/star-01.svg"
                        ></image>
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

                <!-- {#if !fireWorks} -->
                    <defs>
                        <linearGradient id="ellipse" x1="0%" y1="100%" x2="0%" y2="0%">
                            <stop offset="50%" style="stop-color:white;stop-opacity:0" />
                            <stop offset="100%" style="stop-color:white;stop-opacity:0.8" />
                        </linearGradient>
                    </defs>
                    <ellipse
                        style:transition-duration="{endScore/2}ms"
                        cx={width/2}
                        cy={y+(width/22*3.5)}
                        rx={width/22+(y/15)}
                        ry={width/22*4}
                        fill="url(#ellipse)"
                        opacity={fireWorks ? 0 : 1 }
                    ></ellipse>
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
                        }"
                        opacity={fireWorks ? 0 : 1 }></rect>
                    <rect 
                        style:transition-duration="{endScore/2}ms"
                        width={width/25} 
                        height={width/25*2/12} 
                        x={width/2-(width/25/2)} 
                        y={y}
                        fill="#e5e5e5"
                        opacity={fireWorks ? 0 : 1 }></rect>
                    <rect 
                        style:transition-duration="{endScore/2}ms"
                        width={width/25} 
                        height={width/25*2/12} 
                        x={width/2-(width/25/2)} 
                        y={y+(width/25*2)-(width/25*2/12)}
                        fill="#e5e5e5"
                        opacity={fireWorks ? 0 : 1 }></rect>
                <!-- {/if} -->
                <rect
                    style:transition-duration="{endScore/2}ms"
                    width={width/50} 
                    height={height} 
                    x={width/2-(width/50/2)} 
                    y={y+(width/25*2)}
                    fill="white"
                    opacity={fireWorks ? 0 : 1 }>
                    <!-- "#f4bd40" -->
                </rect>
                <!-- {#each Array(8) as _, i}
                    <circle 
                        class="spark"
                        opacity={fireWorks ? 1 : 0}
                        cx={(width/2)+Math.sin(degToRad(360/8*i))*distance} 
                        cy={cy+Math.cos(degToRad(360/8*i))*distance} 
                        r={radius}
                        fill="white">
                    </circle>
                {/each} -->
                {#each Array(num) as _, i}
					<g
						style:opacity={fireWorks ? 1 : 0}
						>
						<circle 
						class="spark"
						cx={(width/2)+Math.sin(degToRad(360/(num/10)*i))
							*(distance*(Math.ceil((i+1)/(num/10))/10))
							+ Math.random()*10} 
						cy={cy+Math.cos(degToRad(360/(num/10)*i))
							*(distance*(Math.ceil((i+1)/(num/10))/10))
							+ Math.random()*10}
						r={fireWorks ? radius : 0}
						fill={
							Math.ceil((i+1)/(num/10)) == 10 ?
							"whitesmoke" : Math.round(Math.random()) == 1 ? 
							selector == 0 ? "#e74b4c" :
                            selector == 1 ? "#51aade" : 
                            selector == 2 ? "#72be44" :
                            selector == 3 ? "#e98a4a" :
                            "#e74b4c" : "#F2EC69"
							}
							>
						</circle>
					</g>
				{/each}
                
            </svg>
            
            <h1 class="text-shadow">your score:<br>
				<span style:font-size="4rem">
					{showScore.toLocaleString("de-DE")}
				</span>
			</h1>

            {#if showRestart}
				<dialog bind:this={leaderBoardDialog}>
					<h1 class="text-shadow">your score:<br>
						<span style:font-size="4rem">
							{showScore.toLocaleString("de-DE")}
						</span>
					</h1>
					{#if Math.round(showScore) > Math.min(...scoreArray) && !submitdisable}
						<div class="input">
							<input type="text" placeholder="your name" bind:value={inputName} disabled={submitdisable}>
							<button class="submit"on:click={insertData} disabled={submitdisable}>Submit</button>
						</div>    
					{/if}
					<div class="leaderboard-list">
						{#each shortLeaderBoard as list, i}
                        <p style:margin="0">
                            <span>{i+1}.</span> 
                            <span>{list.name}</span> 
                            <span>{list.score.toLocaleString("de-DE")}</span> 
                        </p>
						{/each}
					</div>
					<button style:z-index="3" on:click={restart}>RESTART</button>
				</dialog>
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
	@import "$lib/text-shadow.css";

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
	rect, ellipse {
		transition:all cubic-bezier(0.25, 1, 0.5, 1);
	}
	.spark {
		transition:all 500ms cubic-bezier(0.25, 1, 0.5, 1);
	}
	.leaderboard-list {
		font-family:sans-serif;
		display: flex;
		flex-direction: column;
		position: relative;
	}
    .leaderboard-list > p {
        width:90%;
		display: flex;
		justify-content: space-between;
	}
	.leaderboard-list > p > span:nth-child(1) {
		width:10%;
		text-align: left;
	}
	.leaderboard-list > p > span:nth-child(2) {
		flex-grow: 1;
		margin-left:1rem;
		text-align: left;
	}
	.leaderboard-list > p > span:nth-child(3) {
		text-align: right;
	}
	div {
		position: absolute;
		width:100%;
		display: flex;
		justify-content: center;
		align-items: center;
		z-index:-1;
	}
	.input {
		position: relative;
        display:flex;
        justify-content: space-between;
        margin:1rem;
		width:90%;
    }
	.input > button {
        margin: 0;
		width:25%;
        pointer-events: auto;
        cursor: pointer;
        background-color: #efb754;
        color:#051519;
        border-radius: 0.5rem;
        font-size: 1rem;
        font-family: monospace;
	}
	input {
		width:70%;
        pointer-events: auto;
        font-size: 1rem;
        border-radius: 0.5rem;
        padding-block: 0.2rem;
        padding-left: 0.5rem;
        font-family: monospace;
    }
	dialog {
		width:88%;
		max-width: 330px;
		display: flex;
		flex-direction: column;
		z-index:6;
	}
	img {
		width:50%;
		max-width:250px;
	}
	.timer {
		color:black;
		font-size:4rem;
	}
	.warning {
		color:red;
	}
	h3 {
		font-size:2rem;
		margin: 0;
		font-family: 'Luckiest Guy', cursive;
		text-align: center;
		color:darkblue;
	}
	h1 {
		font-size:4rem;
		margin: 0;
		font-family: 'Luckiest Guy', cursive;
		text-align: center;
		color:darkblue;
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