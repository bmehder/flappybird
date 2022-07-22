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
  const PAST_PIPE_PIXEL_THRESHOLD = 5
  const READY_FOR_NEW_PIPE_THRESHOLD = 125

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

  // PREDICATES
  const isBirdPastPipe = pipe => pipe.x === PAST_PIPE_PIXEL_THRESHOLD

  const isReadyForNewPipe = pipe => pipe.x === READY_FOR_NEW_PIPE_THRESHOLD

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

  // DRAW FUNCTIONS
  const drawBackground = ctx => ctx.drawImage(background, 0, 0)

  const drawForeground = (ctx, node) =>
    ctx.drawImage(foreground, 0, node.height - foreground.height)

  const drawBird = ctx => ctx.drawImage(bird, birdX, birdY)

  const drawPipeNorth = (pipe, ctx) => ctx.drawImage(pipeNorth, pipe.x, pipe.y)

  const drawPipeSouth = (pipe, ctx, yOffset) => ctx.drawImage(pipeSouth, pipe.x, pipe.y + yOffset)

  const moveBirdDown = () => (birdY += GRAVITY)

  // OTHER FUNCTIONS
  const startOver = () => window.location.reload()

  const increaseScore = () => {
    score++
    scoreAudio.play()
  }

  const displayScore = (ctx, node) => {
    ctx.fillStyle = '#000'
    ctx.font = '20px Monospace'
    ctx.fillText('Score : ' + score, 10, node.height - 20)
  }

  const makeNewPipe = node => {
    pipes.push({
      x: node.width,
      y: Math.floor(Math.random() * pipeNorth.height) - pipeNorth.height,
    })
  }

  const getPipeOffset = () => pipeNorth.height + GAP

  const movePipeLeft = pipe => pipe.x--

  const init = node => {
    const ctx = node.getContext('2d')

    pipes[0] = {
      x: node.width,
      y: 0,
    }

    const draw = () => {
      const yOffset = getPipeOffset()

      drawBackground(ctx)

      pipes.forEach(pipe => {
        drawPipeNorth(pipe, ctx)
        drawPipeSouth(pipe, ctx, yOffset)
        movePipeLeft(pipe)

        isReadyForNewPipe(pipe) && makeNewPipe(node)

        if (isPipeCollision(pipe, yOffset) || isFloorCollision(node)) {
          startOver()
          return
        }

        isBirdPastPipe(pipe) && increaseScore()
      })

      drawForeground(ctx, node)
      drawBird(ctx)
      moveBirdDown()
      displayScore(ctx, node)

      requestAnimationFrame(draw)
    }
    draw()
  }

  const handleKeydown = e => (birdY -= 25) && flyAudio.play()
</script>

<svelte:window on:keydown={handleKeydown} />

<canvas use:init width="288" height="512" />
