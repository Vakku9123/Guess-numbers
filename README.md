# Guess-numbers
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>🎮 Guess the 4‑Digit Code</title>
  <style>
    /* Scenic background image embedded via base64 (a calm mountain lake) */
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      height: 100vh;
      background: url('https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&w=1950&q=80') no-repeat center center fixed;
      background-size: cover;
      display: flex;
      align-items: center;
      justify-content: center;
      color: white;
    }

    /* Overlay for better text visibility */
    body::before {
      content: "";
      position: fixed;
      top: 0; left: 0; right: 0; bottom: 0;
      background: rgba(0,0,0,0.4);
      z-index: 0;
    }

    .game {
      position: relative;
      z-index: 1;
      background: rgba(0, 0, 0, 0.5);
      padding: 30px 40px;
      border-radius: 12px;
      box-shadow: 0 8px 25px rgba(0,0,0,0.8);
      max-width: 400px;
      width: 90%;
      text-align: center;
      backdrop-filter: blur(5px);
      -webkit-backdrop-filter: blur(5px);
    }

    h2 {
      margin-bottom: 10px;
      font-weight: 700;
      font-size: 2rem;
    }

    p {
      margin: 0 0 20px;
      font-weight: 500;
    }

    input {
      font-size: 2em;
      width: 150px;
      text-align: center;
      padding: 10px;
      border: 2px solid rgba(255, 255, 255, 0.7);
      border-radius: 10px;
      margin: 10px 0;
      background: transparent;
      color: white;
      transition: border-color 0.3s ease;
    }

    input::placeholder {
      color: rgba(255, 255, 255, 0.7);
    }

    input:focus {
      border-color: #66d9ef;
      outline: none;
      box-shadow: 0 0 8px #66d9efaa;
      background: rgba(255,255,255,0.1);
    }

    button {
      padding: 10px 20px;
      font-size: 1em;
      border: none;
      border-radius: 8px;
      margin: 5px;
      cursor: pointer;
      color: white;
      box-shadow: 0 4px 10px rgba(0,0,0,0.4);
      transition: background-color 0.3s ease;
    }

    button#submitBtn {
      background-color: #00b894;
    }

    button#submitBtn:hover {
      background-color: #019875;
    }

    button#resetBtn {
      background-color: #636e72;
    }

    button#resetBtn:hover {
      background-color: #2d3436;
    }

    .result {
      margin-top: 20px;
      font-size: 1.2em;
    }

    .digit {
      display: inline-block;
      font-size: 1.5em;
      width: 30px;
      height: 30px;
      line-height: 30px;
      margin: 4px;
      border-radius: 6px;
      font-weight: bold;
      user-select: none;
    }

    .correct {
      background-color: #00b894;
      color: white;
      box-shadow: 0 0 8px #00b894aa;
    }

    .misplaced {
      background-color: #fdcb6e;
      color: black;
      box-shadow: 0 0 8px #fdcb6eaa;
    }

    .incorrect {
      background-color: #d63031;
      color: white;
      box-shadow: 0 0 8px #d63031aa;
    }

    .message {
      margin-top: 15px;
      font-size: 1.2em;
      font-weight: bold;
      min-height: 1.5em;
    }

    [aria-live] {
      margin-top: 10px;
    }

    @media (max-width: 500px) {
      input {
        width: 120px;
      }
    }
  </style>
</head>
<body>
  <div class="game" role="main" aria-labelledby="title">
    <h2 id="title">🔐 Guess the 4-Digit Code</h2>
    <p>All digits are unique. You have <strong>5</strong> tries.</p>
    <input
      type="text"
      id="guessInput"
      maxlength="4"
      aria-label="Enter 4-digit guess"
      placeholder="1234"
      autocomplete="off"
      inputmode="numeric"
      pattern="[0-9]*"
    />
    <br />
    <button id="submitBtn">Submit</button>
    <button id="resetBtn">Reset</button>
    <div class="message" id="attemptInfo" aria-live="polite"></div>
    <div class="result" id="results" aria-live="polite"></div>
    <div class="message" id="finalMessage" aria-live="polite"></div>
  </div>

  <!-- Embedded click sound (fail attempt) -->
  <audio id="clickSound" preload="auto" src="data:audio/wav;base64,UklGRlgAAABXQVZFZm10IBAAAAABAAEAgD4AAIA+AAABAAgAZGF0YQAAAAA="></audio>

  <!-- Embedded win sound (short ding) -->
  <audio id="winSound" preload="auto" src="data:audio/wav;base64,UklGRjgAAABXQVZFZm10IBAAAAABAAEAESsAACJWAAACABAAZGF0YQgAAA=="></audio>

  <script>
    const secretLength = 4;
    let secret = '';
    let attempts = 0;
    const maxAttempts = 5;
    let gameOver = false;

    const input = document.getElementById('guessInput');
    const submitBtn = document.getElementById('submitBtn');
    const resetBtn = document.getElementById('resetBtn');
    const attemptInfo = document.getElementById('attemptInfo');
    const results = document.getElementById('results');
    const finalMessage = document.getElementById('finalMessage');

    const clickSound = document.getElementById('clickSound');
    const winSound = document.getElementById('winSound');

    function play(sound) {
      sound.currentTime = 0;
      sound.play();
    }

    function generateCode() {
      const digits = '0123456789'.split('');
      let code = '';
      while (code.length < secretLength) {
        const idx = Math.floor(Math.random() * digits.length);
        code += digits.splice(idx, 1);
      }
      return code;
    }

    function startGame() {
      secret = generateCode();
      attempts = 0;
      gameOver = false;
      input.value = '';
      input.disabled = false;
      results.innerHTML = '';
      finalMessage.textContent = '';
      attemptInfo.textContent = `Attempt 0 of ${maxAttempts}`;
      input.focus();
      console.log('Secret code:', secret); // Remove in production
    }

    function validateGuess(guess) {
      return /^\d{4}$/.test(guess) && new Set(guess).size === 4;
    }

    function submitGuess() {
      if (gameOver) return;

      const guess = input.value.trim();
      if (!validateGuess(guess)) {
        alert('Please enter 4 unique digits');
        play(clickSound);
        return;
      }

      attempts++;
      attemptInfo.textContent = `Attempt ${attempts} of ${maxAttempts}`;

      const resultRow = document.createElement('div');

      for (let i = 0; i < secretLength; i++) {
        const span = document.createElement('span');
        span.classList.add('digit');

        if (guess[i] === secret[i]) {
          span.classList.add('correct');
          span.textContent = guess[i];
        } else if (secret.includes(guess[i])) {
          span.classList.add('misplaced');
          span.textContent = guess[i];
        } else {
          span.classList.add('incorrect');
          span.textContent = '*';
        }

        resultRow.appendChild(span);
      }

      results.appendChild(resultRow);
      input.value = '';
      input.focus();

      if (guess === secret) {
        finalMessage.textContent = '🎉 You cracked the code!';
        finalMessage.style.color = '#00ffcc';
        play(winSound);
        gameOver = true;
        input.disabled = true;
      } else if (attempts >= maxAttempts) {
        finalMessage.textContent = `❌ Game Over. The code was ${secret}`;
        finalMessage.style.color = '#ff7675';
        play(clickSound);
        gameOver = true;
        input.disabled = true;
      } else {
        play(clickSound);
      }
    }

    // Submit on Enter key press in input field
    input.addEventListener('keydown', e => {
      if (e.key === 'Enter') {
        submitGuess();
        e.preventDefault();
      }
    });

    submitBtn.addEventListener('click', submitGuess);
    resetBtn.addEventListener('click', startGame);

    // Initialize game on load
    window.onload = startGame;
  </script>
</body>
</html>

