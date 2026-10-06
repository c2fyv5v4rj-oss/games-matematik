# games-matematik
<!DOCTYPE html>
<html lang="ms">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kembara Masa & Waktu - Matematik Tahun 4</title>
  <style>
    :root {
      --primary: #4A90E2;
      --secondary: #FF9F43;
      --success: #2ECC71;
      --danger: #EE5253;
      --bg: #F0F4F8;
      --card-bg: #FFFFFF;
    }

    * {
      box-sizing: border-box;
      font-family: 'Fredoka', 'Comic Sans MS', sans-serif, system-ui;
    }

    body {
      background-color: var(--bg);
      margin: 0;
      padding: 12px;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    .game-card {
      background: var(--card-bg);
      width: 100%;
      max-width: 480px;
      border-radius: 20px;
      box-shadow: 0 10px 25px rgba(0,0,0,0.1);
      padding: 20px;
      text-align: center;
      border: 4px solid #E2E8F0;
    }

    h1 {
      color: var(--primary);
      margin-top: 0;
      font-size: 1.5rem;
    }

    .badge {
      display: inline-block;
      padding: 4px 12px;
      border-radius: 12px;
      font-size: 0.85rem;
      font-weight: bold;
      color: white;
      margin-bottom: 10px;
    }

    .badge-aras1 { background-color: #2ED573; }
    .badge-aras2 { background-color: #FFA502; }
    .badge-aras3 { background-color: #FF4757; }

    .stats-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #F1F2F6;
      padding: 10px 15px;
      border-radius: 12px;
      margin-bottom: 15px;
      font-weight: bold;
      font-size: 1.1rem;
    }

    .lives { color: var(--danger); letter-spacing: 2px; }
    .score { color: var(--primary); }

    .question-box {
      background: #EBF5FF;
      border: 2px dashed var(--primary);
      border-radius: 15px;
      padding: 15px;
      min-height: 90px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 1.15rem;
      color: #2C3E50;
      font-weight: 600;
      margin-bottom: 20px;
    }

    .options-grid {
      display: grid;
      grid-template-columns: 1fr;
      gap: 12px;
    }

    .btn {
      width: 100%;
      min-height: 52px;
      padding: 12px;
      font-size: 1rem;
      font-weight: bold;
      border: none;
      border-radius: 14px;
      cursor: pointer;
      transition: transform 0.1s, background-color 0.2s;
      box-shadow: 0 4px 0 rgba(0,0,0,0.15);
    }

    .btn:active {
      transform: translateY(3px);
      box-shadow: none;
    }

    .btn-option {
      background-color: #F8EFBA;
      color: #2C3E50;
      border: 2px solid #ECCC68;
    }

    .btn-action {
      background-color: var(--primary);
      color: white;
      font-size: 1.1rem;
      margin-top: 10px;
    }

    .input-field {
      width: 100%;
      padding: 12px;
      margin: 8px 0;
      border: 2px solid #CED6E0;
      border-radius: 10px;
      font-size: 1rem;
      box-sizing: border-box;
    }

    .hidden { display: none; }
    .result-score { font-size: 2.5rem; color: var(--primary); margin: 10px 0; }
    .result-percent { font-size: 1.5rem; color: var(--secondary); margin-bottom: 15px; }
  </style>
</head>
<body>

  <div class="game-card">
    <!-- SKRIN MULA -->
    <div id="screen-start">
      <h1>⏳ Cabaran Masa & Waktu</h1>
      <p style="color: #57606F;">Matematik Tahun 4</p>
      <form id="form-start" onsubmit="startGame(event)">
        <input type="text" id="player-name" class="input-field" placeholder="Nama Murid" required>
        <input type="text" id="player-class" class="input-field" placeholder="Kelas (contoh: 4 Cemerlang)" required>
        <button type="submit" class="btn btn-action">Mula Bermain! 🚀</button>
      </form>
    </div>

    <!-- SKRIN PERMAINAN -->
    <div id="screen-game" class="hidden">
      <div class="stats-bar">
        <span class="lives" id="lives-display">❤️❤️❤️</span>
        <span id="level-badge" class="badge">Aras 1</span>
        <span class="score">Markah: <span id="score-display">0</span></span>
      </div>

      <div class="question-box" id="question-text">
        Soalan akan dipaparkan di sini...
      </div>

      <div class="options-grid" id="options-container">
        <!-- Butang jawapan dijana oleh JS -->
      </div>
    </div>

    <!-- SKRIN KEPUTUSAN -->
    <div id="screen-result" class="hidden">
      <h1>🎉 Tamat Permainan!</h1>
      <p id="result-message"></p>
      <div class="result-score"><span id="final-score">0</span> / 10</div>
      <div class="result-percent">Peratus: <span id="final-percent">0</span>%</div>

      <button id="btn-submit" class="btn btn-action" onclick="sendToGoogleSheets()">Hantar Keputusan 📤</button>
      <p id="status-msg" style="margin-top: 10px; font-weight: bold;"></p>
      <button class="btn btn-option" onclick="location.reload()" style="margin-top: 15px;">Main Lagi 🔄</button>
    </div>
  </div>

  <script>
    // Audio Synthesizer (Tanpa perlu fail mp3 luaran)
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(type) {
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.connect(gain);
      gain.connect(audioCtx.destination);

      if (type === 'correct') {
        osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); // D5
        osc.frequency.setValueAtTime(880, audioCtx.currentTime + 0.1); // A5
        gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
        osc.start(); osc.stop(audioCtx.currentTime + 0.3);
      } else if (type === 'wrong') {
        osc.type = 'sawtooth';
        osc.frequency.setValueAtTime(200, audioCtx.currentTime);
        osc.frequency.setValueAtTime(120, audioCtx.currentTime + 0.15);
        gain.gain.setValueAtTime(0.2, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.4);
        osc.start(); osc.stop(audioCtx.currentTime + 0.4);
      }
    }

    // Bank Soalan (10 Soalan / 3 Aras)
    const questions = [
      // Aras 1: Mudah
      { level: 1, badge: 'badge-aras1', text: '1 dekad = ____ tahun.', options: ['5 tahun', '10 tahun', '100 tahun', '1000 tahun'], answer: 1 },
      { level: 1, badge: 'badge-aras1', text: '1 abad = ____ tahun.', options: ['10 tahun', '50 tahun', '100 tahun', '1000 tahun'], answer: 2 },
      { level: 1, badge: 'badge-aras1', text: '2 jam = ____ minit.', options: ['60 minit', '120 minit', '180 minit', '240 minit'], answer: 1 },
      { level: 1, badge: 'badge-aras1', text: '3 minit = ____ saat.', options: ['120 saat', '150 saat', '180 saat', '200 saat'], answer: 2 },
      
      // Aras 2: Sederhana
      { level: 2, badge: 'badge-aras2', text: '2 jam 15 minit + 1 jam 30 minit =', options: ['3 jam 45 minit', '3 jam 30 minit', '4 jam 15 minit', '3 jam 15 minit'], answer: 0 },
      { level: 2, badge: 'badge-aras2', text: '5 hari 8 jam - 2 hari 3 jam =', options: ['3 hari 5 jam', '3 hari 11 jam', '2 hari 5 jam', '7 hari 11 jam'], answer: 0 },
      { level: 2, badge: 'badge-aras2', text: 'Tukarkan 3 dekad 4 tahun kepada tahun.', options: ['34 tahun', '304 tahun', '340 tahun', '30 tahun'], answer: 0 },
      
      // Aras 3: Sukar / KBAT
      { level: 3, badge: 'badge-aras3', text: 'Ali mula belajar pada 2:00 petang dan tamat pada 3:45 petang. Berapakah tempoh masa Ali belajar?', options: ['1 jam 15 minit', '1 jam 30 minit', '1 jam 45 minit', '2 jam'], answer: 2 },
      { level: 3, badge: 'badge-aras3', text: 'Sebuah bas bertolak pada jam 0800 dan tiba selepas 3 jam 30 minit. Waktu bas tiba ialah:', options: ['11:00 pagi', '11:30 pagi', '12:00 tengah hari', '12:30 tengah hari'], answer: 1 },
      { level: 3, badge: 'badge-aras3', text: 'Puan Aminah menjahit 3 helai baju. Masa 1 helai baju ialah 40 minit. Berapakah jumlah masa (jam & minit)?', options: ['1 jam 20 minit', '2 jam', '2 jam 20 minit', '2 jam 40 minit'], answer: 1 }
    ];

    // Pembolehubah Permainan
    let currentQ = 0;
    let score = 0;
    let lives = 3;
    let playerName = "";
    let playerClass = "";

    // Pautan Google Apps Script (Gantikan dengan pautan anda)
    const GOOGLE_SCRIPT_URL = "MASUKKAN_URL_GOOGLE_APPS_SCRIPT_DI_SINI";

    function startGame(e) {
      e.preventDefault();
      playerName = document.getElementById('player-name').value.trim();
      playerClass = document.getElementById('player-class').value.trim();

      if (!playerName || !playerClass) return;

      document.getElementById('screen-start').classList.add('hidden');
      document.getElementById('screen-game').classList.remove('hidden');
      
      loadQuestion();
    }

    function loadQuestion() {
      if (currentQ >= questions.length || lives <= 0) {
        endGame();
        return;
      }

      const q = questions[currentQ];
      
      // Kemaskini Aras Badge
      const badgeElem = document.getElementById('level-badge');
      badgeElem.textContent = `Aras ${q.level}`;
      badgeElem.className = `badge ${q.badge}`;

      document.getElementById('question-text').textContent = `${currentQ + 1}. ${q.text}`;
      
      const optionsContainer = document.getElementById('options-container');
      optionsContainer.innerHTML = '';

      q.options.forEach((opt, index) => {
        const btn = document.createElement('button');
        btn.className = 'btn btn-option';
        btn.textContent = opt;
        btn.onclick = () => checkAnswer(index);
        optionsContainer.appendChild(btn);
      });
    }

    function checkAnswer(selectedIndex) {
      const q = questions[currentQ];
      if (selectedIndex === q.answer) {
        playSound('correct');
        score++;
        document.getElementById('score-display').textContent = score;
      } else {
        playSound('wrong');
        lives--;
        updateLivesDisplay();
      }

      currentQ++;
      if (lives > 0) {
        loadQuestion();
      } else {
        endGame();
      }
    }

    function updateLivesDisplay() {
      let hearts = "";
      for (let i = 0; i < 3; i++) {
        hearts += i < lives ? "❤️" : "🖤";
      }
      document.getElementById('lives-display').textContent = hearts;
    }

    function endGame() {
      document.getElementById('screen-game').classList.add('hidden');
      document.getElementById('screen-result').classList.remove('hidden');

      const percent = Math.round((score / questions.length) * 100);
      document.getElementById('final-score').textContent = score;
      document.getElementById('final-percent').textContent = percent;

      const msgElem = document.getElementById('result-message');
      if (score >= 8) {
        msgElem.textContent = "Tahniah! Anda sangat cemerlang! 🌟";
      } else if (score >= 5) {
        msgElem.textContent = "Syabas! Usaha yang baik! 👍";
      } else {
        msgElem.textContent = "Jangan putus asa, cuba lagi ya! 💪";
      }
    }

    function sendToGoogleSheets() {
      const btnSubmit = document.getElementById('btn-submit');
      const statusMsg = document.getElementById('status-msg');
      const percent = Math.round((score / questions.length) * 100);

      if (GOOGLE_SCRIPT_URL === "MASUKKAN_URL_GOOGLE_APPS_SCRIPT_DI_SINI") {
        statusMsg.style.color = "orange";
        statusMsg.textContent = "⚠️ Sila masukkan URL Google Apps Script dalam kod JS dahulu.";
        return;
      }

      btnSubmit.disabled = true;
      statusMsg.style.color = "blue";
      statusMsg.textContent = "Menghantar data...";

      const payload = {
        nama: playerName,
        kelas: playerClass,
        markah: score,
        peratus: percent + "%"
      };

      fetch(GOOGLE_SCRIPT_URL, {
        method: 'POST',
        mode: 'no-cors',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      })
      .then(() => {
        statusMsg.style.color = "green";
        statusMsg.textContent = "✅ Rekod berjaya dihantar ke Google Sheets!";
      })
      .catch(error => {
        statusMsg.style.color = "red";
        statusMsg.textContent = "❌ Gagal menghantar data. Cuba lagi.";
        btnSubmit.disabled = false;
      });
    }
  </script>
</body>
</html>
