<script>
  const bird = new Image()
  const bg = new Image()
  const fg = new Image()
  const pipeNorth = new Image()
  const pipeSouth = new Image()

  const flyAudio = new Audio()
  const scoreAudio = new Audio()

  bird.src = '/images/bird.png'
  bg.src = '/images/bg.png'
  fg.src = '/images/fg.png'
  pipeNorth.src = '/images/pipeNorth.png'
  pipeSouth.src = '/images/pipeSouth.png'

  flyAudio.src = 'sounds/fly.mp3'
  scoreAudio.src = 'sounds/score.mp3'

  let bY = 150
  let bX = 10
  let gap = 85
  let constant
  let gravity = 1.5
  let score = 0
  var pipe = []

  const init = node => {
    let ctx = node.getContext('2d')

    pipe[0] = {
      x: node.width,
      y: 0,
    }

    function draw() {
      ctx?.drawImage(bg, 0, 0)

      for (var i = 0; i < pipe.length; i++) {
        constant = pipeNorth.height + gap
        ctx?.drawImage(pipeNorth, pipe[i].x, pipe[i].y)
        ctx?.drawImage(pipeSouth, pipe[i].x, pipe[i].y + constant)

        pipe[i].x--

        if (pipe[i].x == 125) {
          pipe.push({
            x: node.width,
            y: Math.floor(Math.random() * pipeNorth.height) - pipeNorth.height,
          })
        }

        // detect collision
        const collision =
          (bX + bird.width >= pipe[i].x &&
            bX <= pipe[i].x + pipeNorth.width &&
            (bY <= pipe[i].y + pipeNorth.height || bY + bird.height >= pipe[i].y + constant)) ||
          bY + bird.height >= node.height - fg.height

        collision && location.reload()

        if (pipe[i].x == 5) {
          score++
          scoreAudio.play()
        }
      }

      ctx.drawImage(fg, 0, node.height - fg.height)

      ctx.drawImage(bird, bX, bY)

      bY += gravity

      ctx.fillStyle = '#000'
      ctx.font = '20px Monospace'
      ctx.fillText('Score : ' + score, 10, node.height - 20)

      requestAnimationFrame(draw)
    }

    draw()
  }

  const handleKeydown = e => {
    bY -= 25
    flyAudio.play()
  }
</script>

<svelte:window on:keydown={handleKeydown} />

<canvas use:init width="288" height="512" />
