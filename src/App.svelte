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
  const PAST_PIPE_THRESHOLD = 5
  const READY_FOR_NEW_PIPE_THRESHOLD = 125
  const BIRD_FLY_OFFSET = 25

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
  let highScore = 0
  let isPaused = false

  // Predicates
  const isBirdPastPipe = i => pipes[i].x === PAST_PIPE_THRESHOLD

  const isReadyForNewPipe = i => pipes[i].x === READY_FOR_NEW_PIPE_THRESHOLD

  const isFloorCollision = node => birdY + bird.height >= node.height - foreground.height

  const isPipeCollision = (i, yOffset) => {
    const isRightOfBirdTouchingLeftOfTopPipe = i => birdX + bird.width >= pipes[i].x

    const isRightOfBirdTouchingBottomOfTopPipe = i => birdX <= pipes[i].x + pipeNorth.width

    const isTopOfBirdTouchingBottomOfTopPipe = i => birdY <= pipes[i].y + pipeNorth.height

    const isBottomOfBirdTouchingTopOfBottomPipe = (i, yOffset) =>
      birdY + bird.height >= pipes[i].y + yOffset

    return (
      isRightOfBirdTouchingLeftOfTopPipe(i) &&
      isRightOfBirdTouchingBottomOfTopPipe(i) &&
      (isTopOfBirdTouchingBottomOfTopPipe(i) || isBottomOfBirdTouchingTopOfBottomPipe(i, yOffset))
    )
  }

  const init = node => {
    const ctx = node.getContext('2d')

    highScore = localStorage.getItem('flappyHighScore') || 0

    pipes[0] = {
      x: node.width,
      y: 0,
    }

    const draw = () => {
      const getYOffeset = () => pipeNorth.height + GAP
      const drawBackground = () => ctx.drawImage(background, 0, 0)
      const drawForeground = () => ctx.drawImage(foreground, 0, node.height - foreground.height)
      const drawBird = () => ctx.drawImage(bird, birdX, birdY)
      const drawPipeNorth = (ctx, i) => ctx.drawImage(pipeNorth, pipes[i].x, pipes[i].y)
      const drawPipeSouth = (ctx, i, yOffset) =>
        ctx.drawImage(pipeSouth, pipes[i].x, pipes[i].y + yOffset)
      const movePipeLeft = i => pipes[i].x--
      const moveBirdDown = () => (birdY += GRAVITY)
      const increaseScore = () => {
        score++
        scoreAudio.play()
      }
      const displayScore = () => {
        ctx.fillStyle = '#000'
        ctx.font = '16px Monospace'
        ctx.fillText(`Score : ${score}`, 10, node.height - 20)
        ctx.fillText(`High Score : ${highScore}`, 140, node.height - 20)
      }
      const setHighScore = () => score > highScore && localStorage.setItem('flappyHighScore', score)

      const makeNewPipe = () => {
        pipes.push({
          x: node.width,
          y: Math.floor(Math.random() * pipeNorth.height) - pipeNorth.height,
        })
      }
      const startOver = () => window.location.reload()

      const yOffset = getYOffeset()

      drawBackground()

      for (let i = 0; i < pipes.length; i++) {
        drawPipeNorth(ctx, i)
        drawPipeSouth(ctx, i, yOffset)
        !isPaused && movePipeLeft(i)

        isReadyForNewPipe(i) && makeNewPipe()

        if (isPipeCollision(i, yOffset) || isFloorCollision(node)) {
          setHighScore()
          startOver()
          return
        }

        isBirdPastPipe(i) && increaseScore()
      }

      drawForeground()
      drawBird()
      !isPaused && moveBirdDown()
      displayScore()

      requestAnimationFrame(draw)
    }
    draw()
  }

  const handleEvent = e => {
    e.key === 'p' && (isPaused = !isPaused)
    birdY -= BIRD_FLY_OFFSET
    flyAudio.play()
  }
</script>

<svelte:window on:keydown={handleEvent} on:click={handleEvent} />

<main>
  <h1>Flappy Bird</h1>
  <canvas use:init width="288" height="512" />
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 1rem;
    width: max-content;
    margin: auto;
    min-height: 100vh;
  }
  main {
    user-select: none;
  }
</style>
