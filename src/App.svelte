<script>
  const bird = new Image()
  const background = new Image()
  const foreground = new Image()
  const pipeNorth = new Image()
  const pipeSouth = new Image()

  const flyAudio = new Audio()
  const scoreAudio = new Audio()

  const pipes = []
  const GRAVITY = 1.5
  const GAP = 85

  bird.src = '/images/bird.png'
  background.src = '/images/bg.png'
  foreground.src = '/images/fg.png'
  pipeNorth.src = '/images/pipeNorth.png'
  pipeSouth.src = '/images/pipeSouth.png'

  flyAudio.src = 'sounds/fly.mp3'
  scoreAudio.src = 'sounds/score.mp3'

  let birdX = 10
  let birdY = 150
  let score = 0

  const isBirdPastPipe = pipe => pipe.x === 5
  const isReadyForNewPipe = pipe => pipe.x === 125

  const isRightOfBirdTouchingLeftOfTopPipe = pipe => birdX + bird.width >= pipe.x
  const isLeftOfBirdTouchingBottomOfTopPipe = pipe => birdX <= pipe.x + pipeNorth.width
  const isTopOfBirdTouchingBottomOfTopPipe = pipe => birdY <= pipe.y + pipeNorth.height
  const isBottomOfBirdTouchingTopOfBottomPipe = (pipe, yOffset) =>
    birdY + bird.height >= pipe.y + yOffset

  const isFloorCollision = node => birdY + bird.height >= node.height - foreground.height
  const isPipeCollision = (pipe, yOffset) =>
    isRightOfBirdTouchingLeftOfTopPipe(pipe) &&
    isLeftOfBirdTouchingBottomOfTopPipe(pipe) &&
    (isTopOfBirdTouchingBottomOfTopPipe(pipe) ||
      isBottomOfBirdTouchingTopOfBottomPipe(pipe, yOffset))

  const drawPipeNorth = (ctx, pipe) => ctx.drawImage(pipeNorth, pipe.x, pipe.y)
  const drawPipeSouth = (ctx, pipe, yOffset) => ctx.drawImage(pipeSouth, pipe.x, pipe.y + yOffset)

  const movePipeLeft = pipe => pipe.x--

  const init = node => {
    const ctx = node.getContext('2d')

    pipes[0] = {
      x: node.width,
      y: 0,
    }

    const draw = () => {
      const yOffset = pipeNorth.height + GAP
      const drawBackground = () => ctx.drawImage(background, 0, 0)
      const drawForeground = () => ctx.drawImage(foreground, 0, node.height - foreground.height)
      const drawBird = () => ctx.drawImage(bird, birdX, birdY)
      const startOver = () => window.location.reload()
      const moveBirdDown = () => (birdY += GRAVITY)
      const displayScore = () => {
        ctx.fillStyle = '#000'
        ctx.font = '20px Monospace'
        ctx.fillText('Score : ' + score, 10, node.height - 20)
      }
      const increaseScore = () => {
        score++
        scoreAudio.play()
      }
      const makeNewPipe = () => {
        pipes.push({
          x: node.width,
          y: Math.floor(Math.random() * pipeNorth.height) - pipeNorth.height,
        })
      }

      drawBackground()

      pipes.forEach(pipe => {
        drawPipeNorth(ctx, pipe)
        drawPipeSouth(ctx, pipe, yOffset)
        movePipeLeft(pipe)

        isReadyForNewPipe(pipe) && makeNewPipe()
        if (isPipeCollision(pipe, yOffset) || isFloorCollision(node)) {
          startOver()
          return
        }
        isBirdPastPipe(pipe) && increaseScore()
      })

      // for (let i = 0; i < pipes.length; i++) {
      //   drawPipeNorth(ctx, i)
      //   drawPipeSouth(ctx, i, yOffset)
      //   movePipeLeft(i)

      //   isReadyForNewPipe(i) && makeNewPipe()
      //   if (isPipeCollision(i, yOffset) || isFloorCollision(node)) {
      //     startOver()
      //     return
      //   }
      //   isBirdPastPipe(i) && increaseScore()
      // }

      drawForeground()
      drawBird()
      moveBirdDown()
      displayScore()

      requestAnimationFrame(draw)
    }
    draw()
  }

  const handleKeydown = e => (birdY -= 25) && flyAudio.play()
</script>

<svelte:window on:keydown={handleKeydown} />

<canvas use:init width="288" height="512" />
