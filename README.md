<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>🐍 Snake Game</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root { --size: 400px; --box: 20px; }
    body{
      margin:0; height:100vh; display:flex; flex-direction:column; align-items:center; justify-content:center;
      background:#111; color:#fff; font-family:Arial, sans-serif;
    }
    h1{ margin:8px 0 4px; }
    #scoreBoard{ margin:6px 0 10px; font-size:18px; }
    .wrap{ position:relative; width:var(--size); height:var(--size); }
    canvas{
      width:var(--size); height:var(--size);
      background:#222; border:2px solid #fff;
      touch-action:none; /* evita scroll durante o swipe */
    }

    /* ====== Joystick invisível (quatro zonas sem sobreposição) ====== */
    #touchZones{ position:absolute; inset:0; display:none; z-index:5; }
    /* ATIVE a linha abaixo para debug visual: troque 0.0 por 0.06 para ver as áreas */
    .zone{ position:absolute; background:rgba(255,255,255,0.0); }

    /* Coordenadas (percentuais) das zonas – sem sobrepor */
    /* Cima: central no topo */
    .zone.up   { left:33%; width:34%; top:0;    height:44%; }
    /* Baixo: central no rodapé */
    .zone.down { left:33%; width:34%; bottom:0; height:44%; }
    /* Esquerda: central na lateral esquerda */
    .zone.left { left:0;   width:44%; top:33%;  height:34%; }
    /* Direita: central na lateral direita */
    .zone.right{ right:0;  width:44%; top:33%;  height:34%; }

    /* Mostra joystick invisível apenas em telas menores (celular) */
    @media (max-width: 768px){
      #touchZones{ display:block; }
      :root{ --size: 92vw; } /* canvas responsivo no mobile */
      canvas{ height:auto; }
    }
  </style>
</head>
<body>
  <h1>🐍 Snake Game</h1>
  <div id="scoreBoard">Pontuação: 0 | Recorde: 0</div>

  <div class="wrap">
    <canvas id="game" width="400" height="400"></canvas>

    <!-- ZONAS DE TOQUE INVISÍVEIS -->
    <div id="touchZones">
      <div class="zone up"    data-dir="UP"></div>
      <div class="zone down"  data-dir="DOWN"></div>
      <div class="zone left"  data-dir="LEFT"></div>
      <div class="zone right" data-dir="RIGHT"></div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");
    const box = 20;

    let snake = [{ x: 9 * box, y: 10 * box }];
    let direction = null;
    let score = 0;
    let record = Number(localStorage.getItem("snakeRecord") || 0);

    // comida inicial
    let food = randFood();

    function randFood(){
      return {
        x: Math.floor(Math.random() * (canvas.width / box)) * box,
        y: Math.floor(Math.random() * (canvas.height / box)) * box
      };
    }

    function updateScore(){
      document.getElementById("scoreBoard").textContent =
        `Pontuação: ${score} | Recorde: ${Math.max(score, record)}`;
    }
    updateScore();

    // ======= TECLADO (PC: setas + WASD) =======
    document.addEventListener("keydown", e => {
      const k = e.key.toLowerCase();
      if ((k === "arrowup"    || k === "w") && direction !== "DOWN")  direction = "UP";
      if ((k === "arrowdown"  || k === "s") && direction !== "UP")    direction = "DOWN";
      if ((k === "arrowleft"  || k === "a") && direction !== "RIGHT") direction = "LEFT";
      if ((k === "arrowright" || k === "d") && direction !== "LEFT")  direction = "RIGHT";
    });

    // ======= SWIPE (gesto) =======
    let startX = 0, startY = 0;
    const start = e => { const t = e.touches[0]; startX = t.clientX; startY = t.clientY; };
    const end   = e => {
      const t = e.changedTouches[0];
      const dx = t.clientX - startX, dy = t.clientY - startY;
      if (Math.abs(dx) > Math.abs(dy)){
        if (dx > 0 && direction !== "LEFT")  direction = "RIGHT";
        if (dx < 0 && direction !== "RIGHT") direction = "LEFT";
      } else {
        if (dy > 0 && direction !== "UP")    direction = "DOWN";
        if (dy < 0 && direction !== "DOWN")  direction = "UP";
      }
    };
    canvas.addEventListener("touchstart", start, {passive:true});
    canvas.addEventListener("touchend",   end,   {passive:true});

    // ======= JOYSTICK INVISÍVEL (quatro zonas) =======
    function setDir(dir){
      if (dir === "UP"    && direction !== "DOWN")  direction = "UP";
      if (dir === "DOWN"  && direction !== "UP")    direction = "DOWN";
      if (dir === "LEFT"  && direction !== "RIGHT") direction = "LEFT";
      if (dir === "RIGHT" && direction !== "LEFT")  direction = "RIGHT";
    }

    document.querySelectorAll("#touchZones .zone").forEach(z => {
      const dir = z.dataset.dir;
      // toque único
      z.addEventListener("touchstart", e => { setDir(dir); }, {passive:true});
      // clique também funciona (para testes no desktop)
      z.addEventListener("click", () => setDir(dir));
    });

    // ======= LÓGICA DO JOGO =======
    function collision(head, body){
      return body.some(p => p.x === head.x && p.y === head.y);
    }

    function draw(){
      // fundo
      ctx.fillStyle = "#222";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // grid
      ctx.strokeStyle = "#333";
      for (let i = 0; i < canvas.width; i += box){
        ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, canvas.height); ctx.stroke();
        ctx.beginPath(); ctx.moveTo(0, i); ctx.lineTo(canvas.width, i); ctx.stroke();
      }

      // comida
      ctx.fillStyle = "red";
      ctx.beginPath();
      ctx.arc(food.x + box/2, food.y + box/2, box/2 - 2, 0, 2*Math.PI);
      ctx.fill();

      // cobra
      for (let i=0;i<snake.length;i++){
        ctx.fillStyle = i===0 ? "#0f0" : "#3c3";
        ctx.fillRect(snake[i].x, snake[i].y, box, box);
        ctx.strokeStyle = "#111";
        ctx.strokeRect(snake[i].x, snake[i].y, box, box);
      }

      // movimento
      let nx = snake[0].x, ny = snake[0].y;
      if (direction === "UP")    ny -= box;
      if (direction === "DOWN")  ny += box;
      if (direction === "LEFT")  nx -= box;
      if (direction === "RIGHT") nx += box;

      // comer
      if (nx === food.x && ny === food.y){
        score++;
        if (score > record){ record = score; localStorage.setItem("snakeRecord", record); }
        updateScore();
        food = randFood();
      } else {
        snake.pop();
      }

      const newHead = {x:nx, y:ny};

      // colisões
      if (nx < 0 || nx >= canvas.width || ny < 0 || ny >= canvas.height || collision(newHead, snake)){
        clearInterval(loop);
        alert("Game Over!");
        return;
      }

      snake.unshift(newHead);
    }

    const loop = setInterval(draw, 120);
  </script>
</body>
</html>

