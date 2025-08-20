<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>🐍 Snake Game</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: #111;
      color: white;
      font-family: Arial, sans-serif;
    }
    #scoreBoard {
      margin: 10px;
      font-size: 18px;
    }
    canvas {
      border: 2px solid #fff;
      background: #222;
    }
  </style>
</head>
<body>
  <h1>🐍 Snake Game</h1>
  <div id="scoreBoard">Pontuação: 0 | Recorde: 0</div>
  <canvas id="game" width="400" height="400"></canvas>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const box = 20;
    let snake = [{ x: 9 * box, y: 10 * box }];
    let direction;
    let score = 0;
    let record = localStorage.getItem("snakeRecord") || 0;

    let food = {
      x: Math.floor(Math.random() * 19 + 1) * box,
      y: Math.floor(Math.random() * 19 + 1) * box
    };

    // ✅ Atualizar placar
    function updateScore() {
      document.getElementById("scoreBoard").textContent =
        "Pontuação: " + score + " | Recorde: " + record;
    }
    updateScore();

    // ✅ Controles no PC (setas + WASD)
    document.addEventListener("keydown", e => {
      if ((e.key === "ArrowLeft" || e.key === "a") && direction !== "RIGHT") direction = "LEFT";
      if ((e.key === "ArrowUp"   || e.key === "w") && direction !== "DOWN")  direction = "UP";
      if ((e.key === "ArrowRight"|| e.key === "d") && direction !== "LEFT")  direction = "RIGHT";
      if ((e.key === "ArrowDown" || e.key === "s") && direction !== "UP")    direction = "DOWN";
    });

    // ✅ Controles no celular (swipe)
    let touchStartX = 0, touchStartY = 0;
    canvas.addEventListener("touchstart", e => {
      let touch = e.touches[0];
      touchStartX = touch.clientX;
      touchStartY = touch.clientY;
    });
    canvas.addEventListener("touchend", e => {
      let touch = e.changedTouches[0];
      let dx = touch.clientX - touchStartX;
      let dy = touch.clientY - touchStartY;

      if (Math.abs(dx) > Math.abs(dy)) {
        if (dx > 0 && direction !== "LEFT") direction = "RIGHT";
        else if (dx < 0 && direction !== "RIGHT") direction = "LEFT";
      } else {
        if (dy > 0 && direction !== "UP") direction = "DOWN";
        else if (dy < 0 && direction !== "DOWN") direction = "UP";
      }
    });

    // ✅ Colisão com corpo
    function collision(head, array) {
      return array.some(part => head.x === part.x && head.y === part.y);
    }

    // ✅ Loop do jogo
    function draw() {
      // Fundo
      ctx.fillStyle = "#222";
      ctx.fillRect(0, 0, 400, 400);

      // Grid de fundo
      ctx.strokeStyle = "#333";
      for (let i = 0; i < 400; i += box) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, 400);
        ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(0, i);
        ctx.lineTo(400, i);
        ctx.stroke();
      }

      // Cobra
      for (let i = 0; i < snake.length; i++) {
        ctx.fillStyle = i === 0 ? "#0f0" : "#3c3";
        ctx.fillRect(snake[i].x, snake[i].y, box, box);
        ctx.strokeStyle = "#111";
        ctx.strokeRect(snake[i].x, snake[i].y, box, box);
      }

      // Comida
      ctx.fillStyle = "red";
      ctx.beginPath();
      ctx.arc(food.x + box/2, food.y + box/2, box/2 - 2, 0, 2 * Math.PI);
      ctx.fill();

      // Movimento
      let snakeX = snake[0].x;
      let snakeY = snake[0].y;

      if (direction === "LEFT") snakeX -= box;
      if (direction === "UP") snakeY -= box;
      if (direction === "RIGHT") snakeX += box;
      if (direction === "DOWN") snakeY += box;

      // Comer comida
      if (snakeX === food.x && snakeY === food.y) {
        score++;
        if (score > record) {
          record = score;
          localStorage.setItem("snakeRecord", record);
        }
        updateScore();
        food = {
          x: Math.floor(Math.random() * 19 + 1) * box,
          y: Math.floor(Math.random() * 19 + 1) * box
        };
      } else {
        snake.pop();
      }

      const newHead = { x: snakeX, y: snakeY };

      // Fim de jogo
      if (
        snakeX < 0 || snakeX >= 400 ||
        snakeY < 0 || snakeY >= 400 ||
        collision(newHead, snake)
      ) {
        clearInterval(game);
        alert("Game Over!");
        return;
      }

      snake.unshift(newHead);
    }

    let game = setInterval(draw, 120);
  </script>
</body>
</html>
