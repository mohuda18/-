<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Simple Pong Game</title>
  <style>
    body {
      background: #1a1a1a;
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      user-select: none;
    }
    #gameCanvas {
      background: #222;
      display: block;
      margin: auto;
      border: 2px solid #fff;
    }
    #scoreboard {
      position: absolute;
      top: 40px;
      left: 50%;
      transform: translateX(-50%);
      color: #fff;
      font-family: Arial, sans-serif;
      font-size: 2rem;
      text-align: center;
      letter-spacing: 2px;
      z-index: 10;
    }
  </style>
</head>
<body>
  <div id="scoreboard">Player: 0 | Computer: 0</div>
  <canvas id="gameCanvas" width="800" height="400"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreboard = document.getElementById("scoreboard");

    // Game constants
    const paddleWidth = 12, paddleHeight = 80;
    const ballSize = 14;
    const initialBallSpeed = 4;
    const paddleSpeed = 7;

    // Game state
    let playerScore = 0, computerScore = 0;
    let leftPaddleY = (canvas.height - paddleHeight) / 2;
    let rightPaddleY = (canvas.height - paddleHeight) / 2;
    let ball = {
      x: canvas.width/2 - ballSize/2,
      y: canvas.height/2 - ballSize/2,
      dx: initialBallSpeed * (Math.random() > 0.5 ? 1 : -1),
      dy: initialBallSpeed * (Math.random() > 0.5 ? 1 : -1)
    };

    // Mouse control for left paddle
    canvas.addEventListener('mousemove', (e) => {
      let rect = canvas.getBoundingClientRect();
      let mouseY = e.clientY - rect.top;
      leftPaddleY = mouseY - paddleHeight/2;
      leftPaddleY = Math.max(0, Math.min(canvas.height - paddleHeight, leftPaddleY));
    });

    // Arrow key control for left paddle
    const keys = { ArrowUp: false, ArrowDown: false };
    window.addEventListener('keydown', e => {
      if (e.key === 'ArrowUp' || e.key === 'ArrowDown') keys[e.key] = true;
    });
    window.addEventListener('keyup', e => {
      if (e.key === 'ArrowUp' || e.key === 'ArrowDown') keys[e.key] = false;
    });

    function resetBall(winner) {
      ball.x = canvas.width/2 - ballSize/2;
      ball.y = canvas.height/2 - ballSize/2;
      ball.dx = initialBallSpeed * (winner === "player" ? -1 : 1) * (Math.random() > 0.5 ? 1 : -1);
      ball.dy = initialBallSpeed * (Math.random() > 0.5 ? 1 : -1);
    }

    function drawEverything() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Draw paddles
      ctx.fillStyle = "#fff";
      ctx.fillRect(0, leftPaddleY, paddleWidth, paddleHeight);
      ctx.fillRect(canvas.width-paddleWidth, rightPaddleY, paddleWidth, paddleHeight);

      // Draw ball
      ctx.beginPath();
      ctx.arc(ball.x + ballSize/2, ball.y + ballSize/2, ballSize/2, 0, Math.PI*2);
      ctx.fill();

      // Net (dashed line in middle)
      ctx.strokeStyle = "#ccc";
      ctx.setLineDash([7, 12]);
      ctx.beginPath();
      ctx.moveTo(canvas.width/2, 0);
      ctx.lineTo(canvas.width/2, canvas.height);
      ctx.stroke();
      ctx.setLineDash([]);
    }

    function moveRightPaddle() {
      const paddleCenter = rightPaddleY + paddleHeight/2;
      if (ball.y + ballSize/2 < paddleCenter - 15)
        rightPaddleY -= paddleSpeed * 0.75;
      else if (ball.y + ballSize/2 > paddleCenter + 15)
        rightPaddleY += paddleSpeed * 0.75;
      // Boundaries
      rightPaddleY = Math.max(0, Math.min(canvas.height - paddleHeight, rightPaddleY));
    }

    function moveLeftPaddle() {
      // Arrow keys movement
      if (keys.ArrowUp) leftPaddleY -= paddleSpeed;
      if (keys.ArrowDown) leftPaddleY += paddleSpeed;
      // Boundaries
      leftPaddleY = Math.max(0, Math.min(canvas.height - paddleHeight, leftPaddleY));
    }

    function updateBall() {
      ball.x += ball.dx;
      ball.y += ball.dy;

      // Wall collision
      if (ball.y <= 0 || ball.y + ballSize >= canvas.height) {
        ball.dy *= -1;
      }

      // Paddle collision (left)
      if (ball.x <= paddleWidth) {
        if (
          ball.y + ballSize > leftPaddleY &&
          ball.y < leftPaddleY + paddleHeight
        ) {
          ball.dx *= -1.05; // increase speed slightly
          // Angle bounce based on contact position
          const hitPos = ((ball.y + ballSize/2) - (leftPaddleY + paddleHeight/2)) / (paddleHeight/2);
          ball.dy += hitPos * 3;
          ball.x = paddleWidth + 1;
        } else if (ball.x <= 0) {
          computerScore++;
          updateScore();
          resetBall("computer");
        }
      }

      // Paddle collision (right)
      if (ball.x + ballSize >= canvas.width - paddleWidth) {
        if (
          ball.y + ballSize > rightPaddleY &&
          ball.y < rightPaddleY + paddleHeight
        ) {
          ball.dx *= -1.05;
          const hitPos = ((ball.y + ballSize/2) - (rightPaddleY + paddleHeight/2)) / (paddleHeight/2);
          ball.dy += hitPos * 3;
          ball.x = canvas.width - paddleWidth - ballSize - 1;
        } else if (ball.x + ballSize >= canvas.width) {
          playerScore++;
          updateScore();
          resetBall("player");
        }
      }

      // Limit ball dy
      ball.dy = Math.max(-8, Math.min(8, ball.dy));
    }

    function updateScore() {
      scoreboard.textContent = `Player: ${playerScore} | Computer: ${computerScore}`;
    }

    function gameLoop() {
      moveLeftPaddle();
      moveRightPaddle();
      updateBall();
      drawEverything();
      requestAnimationFrame(gameLoop);
    }

    // Start game
    updateScore();
    gameLoop();
  </script>
</body>
</html>
