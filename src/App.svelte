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

  const isBirdPastPipe = i => pipes[i].x === 5
  const isReadyForNewPipe = i => pipes[i].x === 125

  const isRightOfBirdTouchingLeftOfTopPipe = i => birdX + bird.width >= pipes[i].x
  const isLeftOfBirdTouchingBottomOfTopPipe = i => birdX <= pipes[i].x + pipeNorth.width
  const isTopOfBirdTouchingBottomOfTopPipe = i => birdY <= pipes[i].y + pipeNorth.height
  const isBottomOfBirdTouchingTopOfBottomPipe = (i, yOffset) =>
    birdY + bird.height >= pipes[i].y + yOffset
  const isFloorCollision = node => birdY + bird.height >= node.height - foreground.height
  const isPipeCollision = (i, yOffset) =>
    isRightOfBirdTouchingLeftOfTopPipe(i) &&
    isLeftOfBirdTouchingBottomOfTopPipe(i) &&
    (isTopOfBirdTouchingBottomOfTopPipe(i) || isBottomOfBirdTouchingTopOfBottomPipe(i, yOffset))

  const drawPipeNorth = (ctx, i) => ctx.drawImage(pipeNorth, pipes[i].x, pipes[i].y)
  const drawPipeSouth = (ctx, i, yOffset) =>
    ctx.drawImage(pipeSouth, pipes[i].x, pipes[i].y + yOffset)

  const movePipeLeft = i => pipes[i].x--

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

      for (let i = 0; i < pipes.length; i++) {
        // const isBirdPastPipe = pipes[i].x === 5
        // const isReadyForNewPipe = pipes[i].x === 125

        // const isRightOfBirdTouchingLeftOfTopPipe = birdX + bird.width >= pipes[i].x
        // const isLeftOfBirdTouchingBottomOfTopPipe = birdX <= pipes[i].x + pipeNorth.width
        // const isTopOfBirdTouchingBottomOfTopPipe = birdY <= pipes[i].y + pipeNorth.height
        // const isBottomOfBirdTouchingTopOfBottomPipe = birdY + bird.height >= pipes[i].y + yOffset
        // const isFloorCollision = birdY + bird.height >= node.height - foreground.height
        // const isPipeCollision =
        //   isRightOfBirdTouchingLeftOfTopPipe &&
        //   isLeftOfBirdTouchingBottomOfTopPipe &&
        //   (isTopOfBirdTouchingBottomOfTopPipe || isBottomOfBirdTouchingTopOfBottomPipe)

        // const drawPipeNorth = () => ctx.drawImage(pipeNorth, pipes[i].x, pipes[i].y)
        // const drawPipeSouth = () => ctx.drawImage(pipeSouth, pipes[i].x, pipes[i].y + yOffset)

        // const movePipeLeft = () => pipes[i].x--

        drawPipeNorth(ctx, i)
        drawPipeSouth(ctx, i, yOffset)
        movePipeLeft(i)

        isReadyForNewPipe(i) && makeNewPipe()
        if (isPipeCollision(i, yOffset) || isFloorCollision(node)) {
          startOver()
          return
        }
        isBirdPastPipe(i) && increaseScore()
      }

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
