<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>MiniTicToc</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    :root {
      --primary: #2563eb;
      --secondary: #1d4ed8;
      --accent: #f59e0b;
      --bg: #f9fafb;
      --cell-bg: #ffffff;
      --text: #1f2937;
      --shadow: rgba(0, 0, 0, 0.05);
      --hover: rgba(37, 99, 235, 0.05);
      --win: rgba(245, 158, 11, 0.2);
    }

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: 'Vazirmatn', sans-serif;
      transition: all 0.2s ease;
    }

    body {
      background: var(--bg);
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
      min-height: 100vh;
      color: var(--text);
    }

    .header {
      text-align: center;
      margin-bottom: 20px;
    }

    h1 {
      font-size: 28px;
      color: var(--primary);
      font-weight: 700;
      margin-bottom: 8px;
    }

    .subtitle {
      font-size: 18px;
      color: #6b7280;
      display: flex;
      align-items: center;
      gap: 8px;
      margin-top: 8px;
    }

    .mode-selector {
      display: flex;
      gap: 12px;
      margin: 15px 0;
    }

    .mode-btn {
      padding: 8px 16px;
      border-radius: 20px;
      font-size: 14px;
      cursor: pointer;
      background: white;
      border: 2px solid transparent;
      font-weight: 500;
    }

    .mode-btn.active {
      background: var(--primary);
      color: white;
      border-color: var(--primary);
    }

    #board {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 12px;
      margin: 20px 0;
      max-width: 320px;
      width: 100%;
    }

    .cell {
      aspect-ratio: 1;
      background: var(--cell-bg);
      border-radius: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 48px;
      font-weight: bold;
      color: var(--text);
      cursor: pointer;
      box-shadow: 0 4px 8px var(--shadow);
      position: relative;
      overflow: hidden;
    }

    .cell:hover {
      background: var(--hover);
    }

    .cell.x::after,
    .cell.o::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      border-radius: 12px;
      z-index: -1;
      opacity: 0;
      transition: opacity 0.3s;
    }

    .cell.x::after {
      background: rgba(37, 99, 235, 0.1);
    }

    .cell.o::after {
      background: rgba(245, 158, 11, 0.1);
    }

    .cell.x:hover::after,
    .cell.o:hover::after {
      opacity: 1;
    }

    .status {
      margin: 15px 0;
      font-size: 18px;
      font-weight: 600;
      color: var(--text);
      text-align: center;
    }

    .winner {
      color: var(--accent);
      animation: pulse 0.8s ease-in-out infinite;
    }

    @keyframes pulse {
      0% { transform: scale(1); }
      50% { transform: scale(1.05); }
      100% { transform: scale(1); }
    }

    button {
      padding: 12px 24px;
      font-size: 16px;
      background: var(--primary);
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 20px;
      font-weight: 600;
      box-shadow: 0 4px 8px rgba(37, 99, 235, 0.2);
    }

    button:hover {
      background: var(--secondary);
      transform: translateY(-2px);
    }

    .win-line {
      position: absolute;
      background: var(--accent);
      height: 4px;
      z-index: 10;
      pointer-events: none;
    }

    .win-line.horizontal {
      top: 50%;
      left: 0;
      right: 0;
      transform: translateY(-50%);
    }

    .win-line.vertical {
      left: 50%;
      top: 0;
      bottom: 0;
      transform: translateX(-50%);
    }

    .win-line.diagonal1 {
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: rotate(45deg);
    }

    .win-line.diagonal2 {
      top: 0;
      right: 0;
      width: 100%;
      height: 100%;
      transform: rotate(-45deg);
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>MiniTicToc</h1>
    <div class="subtitle">🎮 بازی دوز</div>
  </div>

  <div class="mode-selector">
    <button class="mode-btn active" data-mode="two">دو نفره</button>
    <button class="mode-btn" data-mode="ai">یکنفره (هوش مصنوعی)</button>
  </div>

  <div class="status" id="status">نوبت: X</div>

  <div id="board"></div>

  <button id="reset">شروع دوباره</button>

  <script>
    const tg = window.Telegram?.WebApp;
    if (tg) {
      tg.expand();
      tg.disableVerticalSwipes();
    }

    const statusEl = document.getElementById('status');
    const boardEl = document.getElementById('board');
    const resetBtn = document.getElementById('reset');
    const modeButtons = document.querySelectorAll('.mode-btn');

    let currentPlayer = 'X';
    let board = Array(9).fill(null);
    let gameActive = true;
    let currentMode = 'two'; // two or ai
    let winLine = null;

    // ساخت خانه‌ها
    for (let i = 0; i < 9; i++) {
      const cell = document.createElement('div');
      cell.className = 'cell';
      cell.dataset.index = i;
      cell.addEventListener('click', () => handleCellClick(i));
      boardEl.appendChild(cell);
    }

    function handleCellClick(index) {
      if (board[index] !== null || !gameActive) return;

      board[index] = currentPlayer;
      const cell = document.querySelectorAll('.cell')[index];
      cell.textContent = currentPlayer;
      cell.classList.add(currentPlayer.toLowerCase());

      if (checkWin()) {
        gameActive = false;
        statusEl.textContent = `🎉 ${currentPlayer} برنده شد!`;
        statusEl.classList.add('winner');
        drawWinLine();
        return;
      }

      if (board.every(cell => cell !== null)) {
        statusEl.textContent = '🤝 مساوی!';
        gameActive = false;
        return;
      }

      // اگر حالت AI باشد، حرکت کامپیوتر
      if (currentMode === 'ai' && currentPlayer === 'X') {
        setTimeout(() => {
          makeAiMove();
        }, 500);
      } else {
        currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
        statusEl.textContent = `نوبت: ${currentPlayer}`;
        statusEl.classList.remove('winner');
      }
    }

    function makeAiMove() {
      // هوش مصنوعی ساده: اولین خانه خالی
      const emptyCells = board.map((val, idx) => val === null ? idx : null).filter(val => val !== null);
      if (emptyCells.length === 0) return;

      const randomIndex = emptyCells[Math.floor(Math.random() * emptyCells.length)];
      board[randomIndex] = 'O';
      const cell = document.querySelectorAll('.cell')[randomIndex];
      cell.textContent = 'O';
      cell.classList.add('o');

      if (checkWin()) {
        gameActive = false;
        statusEl.textContent = `🎉 O برنده شد!`;
        statusEl.classList.add('winner');
        drawWinLine();
        return;
      }

      if (board.every(cell => cell !== null)) {
        statusEl.textContent = '🤝 مساوی!';
        gameActive = false;
        return;
      }

      currentPlayer = 'X';
      statusEl.textContent = `نوبت: X`;
      statusEl.classList.remove('winner');
    }

    function checkWin() {
      const winPatterns = [
        [0,1,2], [3,4,5], [6,7,8], // سطرها
        [0,3,6], [1,4,7], [2,5,8], // ستون‌ها
        [0,4,8], [2,4,6]           // قطرها
      ];

      for (const pattern of winPatterns) {
        const [a, b, c] = pattern;
        if (board[a] && board[a] === board[b] && board[a] === board[c]) {
          winPattern = pattern;
          return true;
        }
      }
      return false;
    }

    let winPattern = null;

    function drawWinLine() {
      if (!winPattern) return;

      const cells = document.querySelectorAll('.cell');
      const firstCell = cells[winPattern[0]];
      const rect = firstCell.getBoundingClientRect();

      const boardRect = boardEl.getBoundingClientRect();

      const line = document.createElement('div');
      line.className = 'win-line';

      // تعیین نوع خط
      if (winPattern[0] === 0 && winPattern[1] === 1 && winPattern[2] === 2) {
        line.classList.add('horizontal');
        line.style.top = `${rect.top - boardRect.top + rect.height / 2}px`;
        line.style.left = `${rect.left - boardRect.left}px`;
        line.style.width = `${rect.width * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 3 && winPattern[1] === 4 && winPattern[2] === 5) {
        line.classList.add('horizontal');
        line.style.top = `${rect.top - boardRect.top + rect.height / 2}px`;
        line.style.left = `${rect.left - boardRect.left}px`;
        line.style.width = `${rect.width * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 6 && winPattern[1] === 7 && winPattern[2] === 8) {
        line.classList.add('horizontal');
        line.style.top = `${rect.top - boardRect.top + rect.height / 2}px`;
        line.style.left = `${rect.left - boardRect.left}px`;
        line.style.width = `${rect.width * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 0 && winPattern[1] === 3 && winPattern[2] === 6) {
        line.classList.add('vertical');
        line.style.left = `${rect.left - boardRect.left + rect.width / 2}px`;
        line.style.top = `${rect.top - boardRect.top}px`;
        line.style.height = `${rect.height * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 1 && winPattern[1] === 4 && winPattern[2] === 7) {
        line.classList.add('vertical');
        line.style.left = `${rect.left - boardRect.left + rect.width / 2}px`;
        line.style.top = `${rect.top - boardRect.top}px`;
        line.style.height = `${rect.height * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 2 && winPattern[1] === 5 && winPattern[2] === 8) {
        line.classList.add('vertical');
        line.style.left = `${rect.left - boardRect.left + rect.width / 2}px`;
        line.style.top = `${rect.top - boardRect.top}px`;
        line.style.height = `${rect.height * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 0 && winPattern[1] === 4 && winPattern[2] === 8) {
        line.classList.add('diagonal1');
        line.style.top = `${rect.top - boardRect.top}px`;
        line.style.left = `${rect.left - boardRect.left}px`;
        line.style.width = `${rect.width * 3 + 12 * 2}px`;
        line.style.height = `${rect.height * 3 + 12 * 2}px`;
      } else if (winPattern[0] === 2 && winPattern[1] === 4 && winPattern[2] === 6) {
        line.classList.add('diagonal2');
        line.style.top = `${rect.top - boardRect.top}px`;
        line.style.right = `${rect.left - boardRect.left}px`;
        line.style.width = `${rect.width * 3 + 12 * 2}px`;
        line.style.height = `${rect.height * 3 + 12 * 2}px`;
      }

      boardEl.appendChild(line);
      winLine = line;
    }

    resetBtn.addEventListener('click', () => {
      board = Array(9).fill(null);
      currentPlayer = 'X';
      gameActive = true;
      statusEl.textContent = 'نوبت: X';
      statusEl.classList.remove('winner');
      document.querySelectorAll('.cell').forEach(cell => {
        cell.textContent = '';
        cell.classList.remove('x', 'o');
      });
      if (winLine) {
        winLine.remove();
        winLine = null;
      }
      winPattern = null;
    });

    modeButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        modeButtons.forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        currentMode = btn.dataset.mode;
        resetBtn.click(); // ریست بازی بعد از تغییر حالت
      });
    });
  </script>
</body>
</html>
