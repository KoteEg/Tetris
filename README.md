<!DOCTYPE html>
<html>
<head>
  <title>Тетрис</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      background-color: #f0f0f0;
    }

    .game-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      background-color: #fff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
    }

    #game-canvas {
      border: 1px solid #ccc;
      background-color: #ddd;
    }

    .score-container {
      margin-top: 20px;
      font-size: 24px;
      font-weight: bold;
    }

    .controls-container {
      margin-top: 20px;
      display: flex;
      gap: 10px;
    }

    .controls-container button {
      padding: 10px 20px;
      font-size: 16px;
      background-color: #4CAF50;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    .game-over {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.8);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      color: #fff;
      font-size: 24px;
      text-align: center;
    }

    .records-container {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: rgba(0, 0, 0, 0.8);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      color: #fff;
      font-size: 24px;
      text-align: center;
    }

    .records-container ol {
      list-style-type: decimal;
      padding: 0;
      margin: 20px 0;
    }

    .records-container li {
      margin-bottom: 10px;
    }

    .fullscreen-btn {
      position: absolute;
      top: 20px;
      right: 20px;
      padding: 10px 20px;
      font-size: 16px;
      background-color: #4CAF50;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <button class="fullscreen-btn" onclick="toggleFullscreen()">Полноэкранный режим</button>
  <div class="game-container">
    <canvas id="game-canvas" width="800" height="960"></canvas>
    <div class="score-container">
      <p>Счет:</p>
      <span id="score">0</span>
    </div>
    <div class="controls-container">
      <button id="pause-btn" onclick="pauseGame()">Пауза</button>
      <button id="save-btn" onclick="saveGame()">Сохранить игру</button>
      <button id="load-btn" onclick="loadGame()">Загрузить игру</button>
      <button id="records-btn" onclick="showRecords()">Рекорды</button>
    </div>
  </div>
  <div id="game-over-screen" class="game-over" style="display: none;">
    <h2>Игра окончена</h2>
    <p>Ваш счет: <span id="final-score">0</span></p>
    <button onclick="location.reload()">Играть снова</button>
  </div>

  <div class="records-container" style="display: none;">
    <h2>Рекорды</h2>
    <ol id="records-list"></ol>
    <button onclick="hideRecords()">Закрыть</button>
  </div>

  <script>
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');
    let gameInterval;
    let gameSpeed = 500;
    let score = 0;
    let gameOver = false;
    let isPaused = false;
    let records = JSON.parse(localStorage.getItem('tetrisRecords')) || [];

    const shapes = [
      {
        blocks: [
          { x: 0, y: 0 },
          { x: 1, y: 0 },
          { x: 2, y: 0 },
          { x: 3, y: 0 }
        ],
        color: '#00FFFF'
      },
      {

        blocks: [
          { x: 1, y: 0 },
          { x: 0, y: 1 },
          { x: 1, y: 1 },
          { x: 2, y: 1 }
        ],
        color: '#FFA500'
      },
      {
        blocks: [
          { x: 0, y: 0 },
          { x: 0, y: 1 },
          { x: 1, y: 1 },
          { x: 2, y: 1 }
        ],
        color: '#0000FF'
      },
      {
        blocks: [
          { x: 2, y: 0 },
          { x: 0, y: 1 },
          { x: 1, y: 1 },
          { x: 2, y: 1 }
        ],
        color: '#008000'
      },
      {
        blocks: [
          { x: 0, y: 0 },
          { x: 1, y: 0 },
          { x: 1, y: 1 },
          { x: 2, y: 1 }
        ],
        color: '#800080'
      },
      {
        blocks: [
          { x: 1, y: 0 },
          { x: 2, y: 0 },
          { x: 0, y: 1 },
          { x: 1, y: 1 }
        ],
        color: '#FFFF00'
      },
      {
        blocks: [
          { x: 0, y: 0 },
          { x: 1, y: 0 },
          { x: 1, y: 1 },
          { x: 2, y: 1 }
        ],
        color: '#FF0000'
      }
    ];

    let currentShape;
    let currentX = 4;
    let currentY = 0;
    let board = Array.from({ length: 20 }, () => Array(10).fill(0));

    function drawBoard() {
      ctx.fillStyle = '#ddd';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      for (let y = 0; y < 20; y++) {
        for (let x = 0; x < 10; x++) {
          if (board[y][x]) {
            ctx.fillStyle = board[y][x];
            ctx.fillRect(x * 40, y * 40, 40, 40);
          }
        }
      }
    }

    function drawShape() {
      ctx.fillStyle = currentShape.color;
      for (const block of currentShape.blocks) {
        ctx.fillRect((currentX + block.x) * 40, (currentY + block.y) * 40, 40, 40);
      }
    }

    function moveShape(dx, dy) {
      if (!checkCollision(currentX + dx, currentY + dy, currentShape)) {
        currentX += dx;
        currentY += dy;
        drawBoard();
        drawShape();
      }
    }

    function rotateShape() {
      const rotatedBlocks = currentShape.blocks.map(({ x, y }) => ({ x: -y, y: x }));
      if (!checkCollision(currentX, currentY, { blocks: rotatedBlocks, color: currentShape.color })) {
        currentShape.blocks = rotatedBlocks;
        drawBoard();
        drawShape();
      }
    }

    function dropShape() {
      while (!checkCollision(currentX, currentY + 1, currentShape)) {
        currentY++;
      }
      lockShape();
      drawBoard();
      drawShape();
    }

    function checkCollision(x, y, shape) {
      for (const block of shape.blocks) {
        const newX = x + block.x;
        const newY = y + block.y;
        if (newX < 0 || newX >= 10 || newY >= 20 || board[newY][newX]) {
          return true;
        }
      }
      return false;
    }

    function lockShape() {
      for (const block of currentShape.blocks) {
        const newX = currentX + block.x;
        const newY = currentY + block.y;
        board[newY][newX] = currentShape.color;
      }

      clearLines();
      spawnNewShape();
    }

    function clearLines() {
      let linesCleared = 0;
      for (let y = 19; y >= 0; y--) {
        if (board[y].every(cell => cell !== 0)) {
          board.splice(y, 1);
          board.unshift(Array(10).fill(0));
          linesCleared++;
        }
      }

      if (linesCleared > 0) {
        score += Math.pow(2, linesCleared - 1) * 100;
        document.getElementById('score').textContent = score;
        increaseSpeed(linesCleared);
      }
    }

    function spawnNewShape() {
      currentShape = shapes[Math.floor(Math.random() * shapes.length)];
      currentX = 4;
      currentY = 0;

      if (checkCollision(currentX, currentY, currentShape)) {

        gameOver = true;
        document.getElementById('final-score').textContent = score;
        document.getElementById('game-over-screen').style.display = 'flex';
        clearInterval(gameInterval);
        checkAndUpdateRecords();
      } else {
        drawShape();
      }
    }

    function gameLoop() {
      if (!gameOver && !isPaused) {
        if (checkCollision(currentX, currentY + 1, currentShape)) {
          lockShape();
        } else {
          drawBoard();
          currentY++;
          drawShape();
        }
      }
    }

    function startGame() {
      spawnNewShape();
      gameInterval = setInterval(gameLoop, gameSpeed);
    }

    function pauseGame() {
      if (gameOver) return;
      if (isPaused) {
        gameInterval = setInterval(gameLoop, gameSpeed);
        isPaused = false;
        document.getElementById('pause-btn').textContent = 'Пауза';
      } else {
        clearInterval(gameInterval);
        isPaused = true;
        document.getElementById('pause-btn').textContent = 'Продолжить';
      }
    }

    function saveGame() {
      if (gameOver || isPaused) return;
      const gameState = {
        board: board.map(row => [...row]),
        currentShape,
        currentX,
        currentY,
        score,
        gameSpeed,
        records: [...records]
      };
      localStorage.setItem('tetrisGameState', JSON.stringify(gameState));
      alert('Игра сохранена!');
    }

    function loadGame() {
      if (gameOver || isPaused) return;
      const gameState = JSON.parse(localStorage.getItem('tetrisGameState'));
      if (gameState) {
        board = gameState.board.map(row => [...row]);
        currentShape = gameState.currentShape;
        currentX = gameState.currentX;
        currentY = gameState.currentY;
        score = gameState.score;
        gameSpeed = gameState.gameSpeed;
        records = gameState.records;
        drawBoard();
        drawShape();
        document.getElementById('score').textContent = score;
        clearInterval(gameInterval);
        gameInterval = setInterval(gameLoop, gameSpeed);
        alert('Игра загружена!');
      } else {
        alert('Нет сохраненной игры.');
      }
    }

    function showRecords() {
      const recordsList = document.getElementById('records-list');
      recordsList.innerHTML = '';
      records.forEach(({ name, score }) => {
        const li = document.createElement('li');
        li.textContent = `${name}: ${score}`;
        recordsList.appendChild(li);
      });
      document.querySelector('.records-container').style.display = 'flex';
    }

    function hideRecords() {
      document.querySelector('.records-container').style.display = 'none';
    }

    function checkAndUpdateRecords() {
      if (records.length < 10 || score > records[records.length - 1].score) {
        const name = prompt('Введите ваше имя:');
        if (name) {
          records.push({ name, score });
          records.sort((a, b) => b.score - a.score);
          records.splice(10);
          localStorage.setItem('tetrisRecords', JSON.stringify(records));
        }
      }
    }

    function increaseSpeed(linesCleared) {
      gameSpeed = Math.max(100, gameSpeed - linesCleared * 50);
      clearInterval(gameInterval);
      gameInterval = setInterval(gameLoop, gameSpeed);
    }

    function handleKeydown(event) {
      if (gameOver || isPaused) return;

      switch (event.key) {
        case 'a':
        case 'ц':
          moveShape(-1, 0);
          break;
        case 'd':
        case 'ф':
          moveShape(1, 0);
          break;
        case 's':
        case 'ы':
          moveShape(0, 1);
          break;
        case 'w':
        case 'в':

          rotateShape();
          break;
        case ' ':
          dropShape();
          break;
      }

      if (event.key === 'w' || event.key === 'ы') {
        event.preventDefault();
      }
    }

    function handleScroll(event) {
      if (event.deltaY !== 0) {
        window.scrollBy(0, event.deltaY);
      }
    }

    function toggleFullscreen() {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen();
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        }
      }
    }

    window.addEventListener('keydown', handleKeydown);
    window.addEventListener('scroll', handleScroll);
    startGame();
  </script>
</body>
</html>
