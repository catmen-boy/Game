# Game

<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Click Master Game</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      font-family: Arial, sans-serif;
      background-color: #f0f8ff;
    }
    h1 {
      margin-bottom: 20px;
    }
    #score {
      font-size: 2em;
      margin: 10px 0;
    }
    button {
      padding: 10px 20px;
      font-size: 1em;
      margin: 5px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      background-color: #4CAF50;
      color: white;
    }
    #message {
      margin-top: 20px;
      font-size: 1.2em;
      color: #d2691e;
    }
    #cat {
      width: 120px;
      margin-top: 20px;
      transition: transform 0.1s ease;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/@solana/web3.js@1.73.2/lib/index.iife.min.js"></script>
</head>
<body>
  <h1>Click Master</h1>
  <div id="score">Score: 0</div>
  <button onclick="increaseScore()">Click Me!</button>
  <button onclick="resetScore()">Reset</button>
  <div id="message"></div>
  <img id="cat" src="https://cdn.pixabay.com/photo/2017/11/09/21/41/cat-2934720_1280.png" alt="Cat Image">  <script>
    let score = parseInt(localStorage.getItem('score')) || 0;
    const scoreDisplay = document.getElementById('score');
    const messageDisplay = document.getElementById('message');
    const catImage = document.getElementById('cat');

    function updateDisplay() {
      scoreDisplay.innerText = `Score: ${score}`;
      if (score >= 100) {
        messageDisplay.innerText = "You’re a true click master! Sending reward...";
        triggerSolanaReward();
      } else {
        messageDisplay.innerText = "";
      }
    }

    function increaseScore() {
      score++;
      localStorage.setItem('score', score);
      catImage.style.transform = `scale(${1 + score * 0.001})`;
      updateDisplay();
    }

    function resetScore() {
      score = 0;
      localStorage.setItem('score', score);
      updateDisplay();
      catImage.style.transform = 'scale(1)';
    }

    async function triggerSolanaReward() {
      if (!window.solana || !window.solana.isPhantom) {
        messageDisplay.innerText += "\nPhantom wallet not found.";
        return;
      }

      try {
        const connection = new solanaWeb3.Connection(solanaWeb3.clusterApiUrl('devnet'));
        const fromWallet = await window.solana.connect();

        const transaction = new solanaWeb3.Transaction().add(
          solanaWeb3.SystemProgram.transfer({
            fromPubkey: fromWallet.publicKey,
            toPubkey: fromWallet.publicKey, // 실제 보상 주소로 변경 필요
            lamports: 1000 // 예시용: 0.000001 SOL
          })
        );

        let { blockhash } = await connection.getLatestBlockhash();
        transaction.recentBlockhash = blockhash;
        transaction.feePayer = fromWallet.publicKey;

        const signed = await window.solana.signTransaction(transaction);
        const signature = await connection.sendRawTransaction(signed.serialize());
        messageDisplay.innerText += `\nReward sent! TX: ${signature}`;
      } catch (err) {
        messageDisplay.innerText += `\nTransaction failed: ${err.message}`;
      }
    }

    // 초기 표시
    updateDisplay();
  </script></body>
</html>
