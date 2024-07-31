<!DOCTYPE html>
<html>
<head>
  <title>Тетрис</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background-color: #f0f0f0;
      font-family: Arial, sans-serif;
      overflow: hidden;
    }

    .game-container {
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
    }

    canvas {
      border: 1px solid black;
      display: block;
      width: 800px;
      height: 960px;
    }

    p {
      margin: 0.5rem 0;
      font-size: 18px;
    }

    button {
      background-color: #4CAF50;
      border: none;
      color: white;
      padding: 10px 20px;
      text-align: center;
      text-decoration: none;
      display: inline-block;
      font-size: 16px;
      margin: 4px 2px;
      cursor: pointer;
      border-radius: 5px;
    }

    button:hover {
      background-color: #45a049;
    }

    #save-btn, #load-btn {
      background-color: #007bff;
    }

    #save-btn:hover, #load-btn:hover {
      background-color: #0056b3;
    }

    #pause-btn {
      background-color: #f44336;
    }

    #pause-btn:hover {
      background-color: #d32f2f;
    }

    .game-over {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background-color: rgba(0, 0, 0, 0.8);
      color: white;
      font-size: 36px;
      text-align: center;
      z-index: 1;
    }

    .score-container {
      display: flex;
      justify-content: center;
      align-items: center;
      margin-bottom: 10px;
    }

    .score-container span {
      font-size: 24px;
      font-weight: bold;
      margin: 0 10px;
    }
  </style>
</head>
<body>
  <h1>Тетрис</h1>
  <div class="game-container">
    <canvas id="game-canvas" width="800" height="960"></canvas>
    <div class="score-container">
      <p>Счет:</p>
      <span id="score">0</span>
    </div>
    <p>Уровень сложности: 
      <select id="difficulty-select">
        <option value="easy">Легкий</option>
        <option value="medium">Средний</option>
        <option value="hard">Сложный</option>
      </select>
    </p>
    <div>
      <button id="fullscreen-btn" onclick="toggleFullscreen()">Полноэкранный режим</button>
      <button id="save-btn">Сохранить игру</button>
      <button id="load-btn">Загрузить игру</button>
      <button id="pause-btn">Пауза</button>
    </div>
  </div>
  <div id="game-over-screen" class="game-over" style="display: none; position: fixed; top: 0; left: 0; background-color: rgba(0, 0, 0, 0.8);">
    <h2>Игра окончена</h2>
    <p>Ваш счет: <span id="final-score">0</span></p>
    <button onclick="location.reload()">Играть снова</button>
  </div>

  <script>
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');
    const scoreElement = document.getElementById('score');
    const finalScoreElement = document.getElementById('final-score');
    const difficultySelect = document.getElementById('difficulty-select');
    const gameOverScreen = document.getElementById('game-over-screen');
    const pauseBtn = document.getElementById('pause-btn');

    const BLOCK_SIZE = 40;
    const GRID_WIDTH = 20;
    const GRID_HEIGHT = 24;

    let gameInterval;
    let score = 0;
    let gameOver = false;
    let currentShape;
    let currentShapeX;

    let currentShapeY;

    let grid = [];
    let isPaused = false;

    const shapes = [
      { color: 'yellow', blocks: [[0, 0], [0, 1], [1, 0], [1, 1]] },
      { color: 'cyan', blocks: [[0, 0], [0, 1], [0, 2], [0, 3]] },
      { color: 'purple', blocks: [[0, 0], [0, 1], [0, 2], [1, 1]] },
      { color: 'green', blocks: [[0, 0], [0, 1], [1, 1], [1, 2]] },
      { color: 'red', blocks: [[0, 0], [0, 1], [1, 0], [1, 1]] },
      { color: 'orange', blocks: [[0, 1], [0, 2], [1, 0], [1, 1]] },
      { color: 'blue', blocks: [[0, 0], [0, 1], [1, 1], [1, 2]] }
    ];

    function drawBlock(x, y, color) {
      ctx.fillStyle = color;
      ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
      ctx.strokeStyle = 'black';
      ctx.strokeRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
    }

    function drawGrid() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      for (let x = 0; x < GRID_WIDTH; x++) {
        for (let y = 0; y < GRID_HEIGHT; y++) {
          if (grid[y][x]) {
            drawBlock(x, y, grid[y][x]);
          } else {
            drawBlock(x, y, '#ccc');
          }
        }
      }

      if (currentShape) {
        for (let i = 0; i < currentShape.blocks.length; i++) {
          const [x, y] = currentShape.blocks[i];
          drawBlock(currentShapeX + x, currentShapeY + y, currentShape.color);
        }
      }
    }

    function updateScore() {
      scoreElement.textContent = score;
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
        case 'Escape':
          pauseGame();
          break;
      }

      // Предотвращаем скроллинг страницы
      if (event.key === 'w' || event.key === 'ы') {
        event.preventDefault();
      }
    }

    function handleScroll(event) {
      // Проверяем, находимся ли мы в полноэкранном режиме
      if (document.fullscreenElement) {
        event.preventDefault();
      } else {
        // Позволяем использовать колесико для прокрутки страницы
        if (event.deltaY !== 0) {
          window.scrollBy(0, event.deltaY);
        }
      }
    }

    function moveShape(dx, dy) {
      if (!checkCollision(currentShapeX + dx, currentShapeY + dy, currentShape)) {
        currentShapeX += dx;
        currentShapeY += dy;
        drawGrid();
      }
    }

    function rotateShape() {
      const rotatedShape = {
        color: currentShape.color,
        blocks: currentShape.blocks.map(([x, y]) => [-y, x])
      };
      if (!checkCollision(currentShapeX, currentShapeY, rotatedShape)) {
        currentShape = rotatedShape;
        drawGrid();
      }
    }

    function dropShape() {
      while (!checkCollision(currentShapeX, currentShapeY + 1, currentShape)) {
        currentShapeY++;
      }
      fixShape();
      clearLines();
      currentShape = null;
      spawnNewShape();
      drawGrid();
    }

    function checkCollision(x, y, shape) {
      for (let i = 0; i < shape.blocks.length; i++) {
        const [dx, dy] = shape.blocks[i];
        const newX = x + dx;
        const newY = y + dy;
        if (
          newX < 0 ||
          newX >= GRID_WIDTH ||
          newY >= GRID_HEIGHT ||
          (newY >= 0 && grid[newY][newX])
        ) {
          return true;
        }
      }

      return false;
    }

    function fixShape() {
      for (let i = 0; i < currentShape.blocks.length; i++) {
        const [x, y] = currentShape.blocks[i];
        grid[currentShapeY + y][currentShapeX + x] = currentShape.color;
      }
    }

    function clearLines() {
      let linesCleared = 0;
      const rowsToAnimate = [];

      for (let y = GRID_HEIGHT - 1; y >= 0; y--) {
        if (grid[y].every(cell => cell)) {
          rowsToAnimate.push(y);
          grid.splice(y, 1);
          grid.unshift(Array(GRID_WIDTH).fill(null));
          linesCleared++;
        }
      }

      if (linesCleared > 0) {
        score += 100 * linesCleared;
        updateScore();
        animateLines(rowsToAnimate);
      }
    }

    function animateLines(rows) {
      let opacity = 1;
      const interval = setInterval(() => {
        opacity -= 0.1;
        drawGrid();
        rows.forEach(y => {
          for (let x = 0; x < GRID_WIDTH; x++) {
            ctx.fillStyle = `rgba(255, 255, 255, ${opacity})`;
            ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
          }
        });

        if (opacity <= 0) {
          clearInterval(interval);
        }
      }, 50);
    }

    function spawnNewShape() {
      currentShape = shapes[Math.floor(Math.random() * shapes.length)];
      currentShapeX = Math.floor(Math.random() * (GRID_WIDTH - currentShape.blocks.length));
      currentShapeY = 0;

      if (checkCollision(currentShapeX, currentShapeY, currentShape)) {
        gameOver = true;
      }
    }

    function gameLoop() {
      if (gameOver) {
        clearInterval(gameInterval);
        gameOverScreen.style.display = 'flex';
        finalScoreElement.textContent = score;
        return;
      }

      if (!currentShape) {
        spawnNewShape();
      }

      moveShape(0, 1);

      if (checkCollision(currentShapeX, currentShapeY + 1, currentShape)) {
        fixShape();
        clearLines();
        currentShape = null;
        spawnNewShape();
      }

      drawGrid();
    }

    function setGameSpeed() {
      switch (difficultySelect.value) {
        case 'medium':
          gameSpeed = 200;
          break;
        case 'hard':
          gameSpeed = 100;
          break;
        default:
          gameSpeed = 300;
      }
      clearInterval(gameInterval);
      gameInterval = setInterval(gameLoop, gameSpeed);
    }

    function startGame() {
      for (let y = 0; y < GRID_HEIGHT; y++) {
        grid[y] = Array(GRID_WIDTH).fill(null);
      }
      setGameSpeed();
    }

    function saveGame() {
      const gameState = {
        score,
        grid,
        currentShapeX,
        currentShapeY,
        currentShape,
        isPaused
      };
      localStorage.setItem('tetrisGame', JSON.stringify(gameState));
    }

    function loadGame() {
      const gameState = JSON.parse(localStorage.getItem('tetrisGame'));
      if (gameState) {
        score = gameState.score;
        grid = gameState.grid;
        currentShapeX = gameState.currentShapeX;
        currentShapeY = gameState.currentShapeY;
        currentShape = gameState.currentShape;
        isPaused = gameState.isPaused;
        updateScore();
        drawGrid();
        if (isPaused) {
          pauseGame();
        } else {
          setGameSpeed();
        }
      }
    }

    function pauseGame() {
      if (isPaused) {
        setGameSpeed();
        isPaused = false;
        pauseBtn.textContent = 'Пауза';
      } else {
        clearInterval(gameInterval);
        isPaused = true;
        pauseBtn.textContent = 'Продолжить';
      }
    }

    function toggleFullscreen() {
      if (!document.fullscreenElement) {

        document.documentElement.requestFullscreen();
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        drawGrid();
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen();
          canvas.width = 800;
          canvas.height = 960;
          drawGrid();
        }
      }
    }

    window.addEventListener('resize', () => {
      canvas.width = canvas.offsetWidth;
      canvas.height = canvas.offsetHeight;
      drawGrid();
    });

    document.addEventListener('keydown', handleKeydown);
    document.addEventListener('wheel', handleScroll);
    difficultySelect.addEventListener('change', setGameSpeed);
    document.getElementById('save-btn').addEventListener('click', saveGame);
    document.getElementById('load-btn').addEventListener('click', loadGame);
    document.getElementById('pause-btn').addEventListener('click', pauseGame);
    document.getElementById('fullscreen-btn').addEventListener('click', toggleFullscreen);

    startGame();
  </script>
</body>
</html>
