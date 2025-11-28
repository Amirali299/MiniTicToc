<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>بازی دوز</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; font-family: Vazirmatn, sans-serif; }
    body {
      background: #f5f5f5;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
      min-height: 100vh;
    }
    h1 {
      margin: 15px 0;
      color: #333;
      font-size: 24px;
    }
    #status {
      margin: 10px 0;
      font-size: 18px;
      font-weight: bold;
      color: #e74c3c;
    }
    #board {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 8px;
      margin: 20px 0;
      max-width: 300px;
      width: 100%;
    }
    .cell {
      aspect-ratio: 1;
      background: white;
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 48px;
      font-weight: bold;
      color: #2c3e50;
      cursor: pointer;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      transition: background 0.2s;
    }
    .cell:hover {
      background: #f0f0f0;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      background: #3498db;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      margin-top: 10px;
    }
    button:hover {
      background: #2980b9;
    }
  </style>
</head>
<body>
  <h1>بازی دوز 🎮</h1>
  <div id="status">نوبت: X</div>
  <div id="board"></div>
  <button id="reset">شروع دوباره</button>

  <script>
    // فقط برای اینکه مینی‌اپ در تلگرام کامل نمایش داده بشه
    const tg = window.Telegram?.WebApp;
    if (tg) {
      tg.expand();
      tg.disableVerticalSwipes(); // جلوگیری از بسته شدن تصادفی
    }

    const statusEl = document.getElementById('status');
    const boardEl = document.getElementById('board');
    const resetBtn = document.getElementById('reset');

    let currentPlayer = 'X';
    let board = Array(9).fill(null);
    let gameActive = true;

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
      document.querySelectorAll('.cell')[index].textContent = currentPlayer;

      if (checkWin()) {
        statusEl.textContent = `🎉 ${currentPlayer} برنده شد!`;
        gameActive = false;
        return;
      }

      if (board.every(cell => cell !== null)) {
        statusEl.textContent = '🤝 مساوی!';
        gameActive = false;
        return;
      }

      currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
      statusEl.textContent = `نوبت: ${currentPlayer}`;
    }

    function checkWin() {
      const winPatterns = [
        [0,1,2], [3,4,5], [6,7,8], // سطرها
        [0,3,6], [1,4,7], [2,5,8], // ستون‌ها
        [0,4,8], [2,4,6]           // قطرها
      ];
      return winPatterns.some(pattern => {
        const [a, b, c] = pattern;
        return board[a] && board[a] === board[b] && board[a] === board[c];
      });
    }

    resetBtn.addEventListener('click', () => {
      board = Array(9).fill(null);
      currentPlayer = 'X';
      gameActive = true;
      statusEl.textContent = 'نوبت: X';
      document.querySelectorAll('.cell').forEach(cell => cell.textContent = '');
    });
  </script>
</body>
</html>
