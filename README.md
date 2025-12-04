<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8" />
  <title>Flappy Bird Clone</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    body {
      background: radial-gradient(circle at top, #1e3c72, #13151a);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #f5f5f5;
    }

    .game-wrapper {
      background: #111827;
      border-radius: 18px;
      padding: 16px 18px 22px;
      box-shadow: 0 18px 35px rgba(0, 0, 0, 0.6);
      text-align: center;
    }

    h1 {
      font-size: 20px;
      margin-bottom: 4px;
    }

    .subtitle {
      font-size: 12px;
      opacity: 0.8;
      margin-bottom: 10px;
    }

    canvas {
      display: block;
      background: linear-gradient(#6dd5fa, #2980b9);
      border-radius: 14px;
      border: 2px solid rgba(255, 255, 255, 0.08);
    }

    .hint {
      margin-top: 10px;
      font-size: 12px;
      opacity: 0.9;
    }

    .hint span {
      font-weight: 600;
    }
  </style>
</head>
<body>
  <div class="game-wrapper">
    <h1>Flappy Bird</h1>
    <div class="subtitle">Klik of druk op spatie om te vliegen</div>
    <canvas id="gameCanvas" width="400" height="600"></canvas>
    <div class="hint">
      <span>Besturing:</span> Spatie / klik / tik ·
      <span>Restart:</span> klik of spatie bij "Game Over"
    </div>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    /** @type {CanvasRenderingContext2D} */
    const ctx = canvas.getContext("2d");

    const WIDTH = canvas.width;
    const HEIGHT = canvas.height;

    // Game state
    let gameState = "start"; // "start" | "playing" | "gameover"
    let score = 0;
    let bestScore = 0;

    // Bird
    const bird = {
      x: 80,
      y: HEIGHT / 2,
      radius: 16,
      velocity: 0,
      gravity: 0.45,
      jumpStrength: -7.5
    };

    // Pipes
    let pipes = [];
    const pipeWidth = 70;
    const pipeGap = 150;
    const pipeSpeed = 2.6;
    const pipeSpawnDistance = 220;

    function resetGame() {
      score = 0;
      bird.y = HEIGHT / 2;
      bird.velocity = 0;
      pipes = [];
      spawnPipe();
      gameState = "start";
    }

    function spawnPipe() {
      const minTopHeight = 50;
      const maxTopHeight = HEIGHT - pipeGap - 120;
      const topHeight =
        Math.random() * (maxTopHeight - minTopHeight) + minTopHeight;

      pipes.push({
        x: WIDTH + 20,
        top: topHeight,     // hoogte van de bovenste pijp
        gap: pipeGap,
        passed: false
      });
    }

    function flap() {
      if (gameState === "start") {
        gameState = "playing";
        bird.velocity = bird.jumpStrength;
      } else if (gameState === "playing") {
        bird.velocity = bird.jumpStrength;
      } else if (gameState === "gameover") {
        resetGame();
      }
    }

    // Input handlers
    window.addEventListener("keydown", (e) => {
      if (e.code === "Space") {
        e.preventDefault();
        flap();
      }
    });

    canvas.addEventListener("mousedown", () => flap());
    canvas.addEventListener("touchstart", (e) => {
      e.preventDefault();
      flap();
    });

    function update() {
      if (gameState === "playing") {
        // Bird physics
        bird.velocity += bird.gravity;
        bird.y += bird.velocity;

        // Spawn pipes
        if (pipes.length === 0) {
          spawnPipe();
        } else {
          const lastPipe = pipes[pipes.length - 1];
          if (lastPipe.x < WIDTH - pipeSpawnDistance) {
            spawnPipe();
          }
        }

        // Update pipes
        pipes.forEach((pipe) => {
          pipe.x -= pipeSpeed;
        });

        // Remove off-screen pipes
        pipes = pipes.filter((pipe) => pipe.x + pipeWidth > -10);

        // Score update (als vogel voorbij pijp is)
        pipes.forEach((pipe) => {
          if (!pipe.passed && pipe.x + pipeWidth < bird.x - bird.radius) {
            pipe.passed = true;
            score++;
            if (score > bestScore) bestScore = score;
          }
        });

        // Collision met bodem of plafond
        if (
          bird.y + bird.radius > HEIGHT ||
          bird.y - bird.radius < 0
        ) {
          gameOver();
        }

        // Collision met pijpen
        for (const pipe of pipes) {
          const inXRange =
            bird.x + bird.radius > pipe.x &&
            bird.x - bird.radius < pipe.x + pipeWidth;
          if (inXRange) {
            const topPipeBottom = pipe.top;
            const bottomPipeTop = pipe.top + pipe.gap;

            if (bird.y - bird.radius < topPipeBottom ||
                bird.y + bird.radius > bottomPipeTop) {
              gameOver();
              break;
            }
          }
        }
      } else if (gameState === "start") {
        // Klein "zweef" effect
        bird.y += Math.sin(Date.now() / 200) * 0.5;
      } else if (gameState === "gameover") {
        // Vogel rustig laten vallen
        if (bird.y + bird.radius < HEIGHT) {
          bird.velocity += bird.gravity * 0.4;
          bird.y += bird.velocity;
        }
      }
    }

    function gameOver() {
      gameState = "gameover";
      bird.velocity = 0;
    }

    function drawBackground() {
      // Lucht wordt al gezet via canvas background, maar we voegen grond toe
      const groundHeight = 80;

      // Grond
      ctx.fillStyle = "#2c3e50";
      ctx.fillRect(0, HEIGHT - groundHeight, WIDTH, groundHeight);

      // Lijn boven de grond
      ctx.strokeStyle = "rgba(255,255,255,0.25)";
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(0, HEIGHT - groundHeight);
      ctx.lineTo(WIDTH, HEIGHT - groundHeight);
      ctx.stroke();

      // Een paar simpele 'bergen' / heuvels
      ctx.fillStyle = "#1f2933";
      ctx.beginPath();
      ctx.moveTo(-40, HEIGHT - groundHeight);
      ctx.quadraticCurveTo(
        WIDTH * 0.25,
        HEIGHT - groundHeight - 60,
        WIDTH * 0.6,
        HEIGHT - groundHeight
      );
      ctx.lineTo(WIDTH, HEIGHT);
      ctx.lineTo(0, HEIGHT);
      ctx.closePath();
      ctx.fill();
    }

    function drawPipes() {
      pipes.forEach((pipe) => {
        const topHeight = pipe.top;
        const gap = pipe.gap;

        // Bovenste pijp
        ctx.fillStyle = "#14532d";
        ctx.fillRect(pipe.x, 0, pipeWidth, topHeight);
        // Rand
        ctx.fillStyle = "#166534";
        ctx.fillRect(pipe.x - 2, topHeight - 20, pipeWidth + 4, 20);

        // Onderste pijp
        const bottomY = topHeight + gap;
        ctx.fillStyle = "#14532d";
        ctx.fillRect(pipe.x, bottomY, pipeWidth, HEIGHT - bottomY);
        // Rand
        ctx.fillStyle = "#166534";
        ctx.fillRect(pipe.x - 2, bottomY, pipeWidth + 4, 20);
      });
    }

    function drawBird() {
      // Schaduw
      ctx.save();
      ctx.globalAlpha = 0.3;
      ctx.beginPath();
      ctx.ellipse(
        bird.x + 4,
        bird.y + bird.radius + 4,
        bird.radius * 1.2,
        bird.radius * 0.5,
        0,
        0,
        Math.PI * 2
      );
      ctx.fillStyle = "#000000";
      ctx.fill();
      ctx.restore();

      // Lichaam
      ctx.beginPath();
      ctx.arc(bird.x, bird.y, bird.radius, 0, Math.PI * 2);
      ctx.fillStyle = "#facc15";
      ctx.fill();
      ctx.lineWidth = 2;
      ctx.strokeStyle = "#92400e";
      ctx.stroke();

      // Vleugel
      ctx.beginPath();
      ctx.ellipse(
        bird.x - 4,
        bird.y,
        bird.radius * 0.7,
        bird.radius * 0.5,
        -0.7,
        0,
        Math.PI * 2
      );
      ctx.fillStyle = "#fbbf24";
      ctx.fill();

      // Oog
      ctx.beginPath();
      ctx.arc(bird.x + 6, bird.y - 4, bird.radius * 0.3, 0, Math.PI * 2);
      ctx.fillStyle = "#ffffff";
      ctx.fill();
      ctx.beginPath();
      ctx.arc(bird.x + 8, bird.y - 4, bird.radius * 0.15, 0, Math.PI * 2);
      ctx.fillStyle = "#0f172a";
      ctx.fill();

      // Snavel
      ctx.beginPath();
      ctx.moveTo(bird.x + bird.radius, bird.y + 2);
      ctx.lineTo(bird.x + bird.radius + 10, bird.y + 6);
      ctx.lineTo(bird.x + bird.radius, bird.y + 10);
      ctx.closePath();
      ctx.fillStyle = "#f97316";
      ctx.fill();
    }

    function drawText() {
      // Score rechtsboven
      ctx.font = "bold 26px system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI'";
      ctx.fillStyle = "rgba(255,255,255,0.95)";
      ctx.textAlign = "right";
      ctx.fillText(score.toString(), WIDTH - 16, 40);

      ctx.font = "12px system-ui";
      ctx.fillStyle = "rgba(255,255,255,0.6)";
      ctx.fillText("Best: " + bestScore, WIDTH - 16, 58);

      ctx.textAlign = "center";

      if (gameState === "start") {
        ctx.font = "20px system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI'";
        ctx.fillStyle = "rgba(255,255,255,0.95)";
        ctx.fillText("Klik of druk op spatie", WIDTH / 2, HEIGHT / 2 - 30);

        ctx.font = "14px system-ui";
        ctx.fillStyle = "rgba(255,255,255,0.8)";
        ctx.fillText("Om te starten: tik / klik / spatie", WIDTH / 2, HEIGHT / 2);
      }

      if (gameState === "gameover") {
        ctx.font = "26px system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI'";
        ctx.fillStyle = "rgba(255,255,255,0.98)";
        ctx.fillText("Game Over", WIDTH / 2, HEIGHT / 2 - 40);

        ctx.font = "16px system-ui";
        ctx.fillStyle = "rgba(255,255,255,0.9)";
        ctx.fillText("Score: " + score, WIDTH / 2, HEIGHT / 2 - 10);
        ctx.fillText("Best: " + bestScore, WIDTH / 2, HEIGHT / 2 + 18);

        ctx.font = "13px system-ui";
        ctx.fillStyle = "rgba(255,255,255,0.8)";
        ctx.fillText(
          "Klik of druk op spatie om opnieuw te spelen",
          WIDTH / 2,
          HEIGHT / 2 + 46
        );
      }
    }

    function draw() {
      ctx.clearRect(0, 0, WIDTH, HEIGHT);
      drawBackground();
      drawPipes();
      drawBird();
      drawText();
    }

    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    // Start
    resetGame();
    gameLoop();
  </script>
</body>
</html>
