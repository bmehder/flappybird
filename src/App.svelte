<script>
	import { onMount } from 'svelte'
	let cvs

	let bY = 150
	
	const fly = new Audio()
	
	
	onMount(() => {		
		let ctx = cvs?.getContext("2d")

		const bird = new Image()
		const bg = new Image()
		const fg = new Image()
		const pipeNorth = new Image()
		const pipeSouth = new Image() 

		bird.src = "https://mehder.com/flappybird/images/bird.png"
		bg.src = "https://mehder.com/flappybird/images/bg.png"
		fg.src = "https://mehder.com/flappybird/images/fg.png"
		pipeNorth.src = "https://mehder.com/flappybird/images/pipeNorth.png"
		pipeSouth.src = "https://mehder.com/flappybird/images/pipeSouth.png"

		const scor = new Audio()

		fly.src = "sounds/fly.mp3"
		scor.src = "sounds/score.mp3"

		let gap = 85
		let constant

		let bX = 10

		let gravity = 1.5

		let score = 0

		var pipe = []

		pipe[0] = {
				x : cvs?.width,
				y : 0
		}

		function draw() {
			ctx?.drawImage(bg,0,0)

			for(var i = 0; i < pipe.length; i++) {

				constant = pipeNorth.height+gap;
				ctx?.drawImage(pipeNorth,pipe[i].x,pipe[i].y)
				ctx?.drawImage(pipeSouth,pipe[i].x,pipe[i].y+constant)

				pipe[i].x--

				if ( pipe[i].x == 125 ){
						pipe.push({
								x : cvs.width,
								y : Math.floor(Math.random()*pipeNorth.height)-pipeNorth?.height
						})
				}

				// detect collision

				if ( bX + bird.width >= pipe[i].x && bX <= pipe[i].x + pipeNorth.width && (bY <= pipe[i].y + pipeNorth.height || bY+bird.height >= pipe[i].y+constant) || bY + bird.height >=  cvs.height - fg.height) {
						location.reload() // reload the page
				}

				if (pipe[i].x == 5) {
						score++
						scor.play()
				}
			}

			ctx.drawImage(fg,0,cvs.height - fg.height);

			ctx.drawImage(bird,bX,bY);

			bY += gravity;

			ctx.fillStyle = "#000";
			ctx.font = "20px Verdana";
			ctx.fillText("Score : "+score,10,cvs.height-20);

			requestAnimationFrame(draw);
		}

		draw()
	}) 
	
	const handleKeydown = e => {
		console.log('yp')
		bY -= 25;
//     fly.play();
	} 
</script>

<svelte:window on:keydown={handleKeydown} />
   
<canvas bind:this="{cvs}" id="canvas" width="288" height="512" />

<style>
	:global(body) {
		height: 100vh;
	}
	canvas {
		border: 1px solid red
	}
</style>