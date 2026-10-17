<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Cyberpunk 駭客攻防戰 (75張牌組微調版)</title>
  <!-- 引入 PeerJS 函式庫 -->
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #0d0f17;
      color: #00ffcc;
      margin: 0;
      padding: 20px;
    }
    h1, h2, h3, h4 { margin: 5px 0; text-shadow: 0 0 5px #00ffcc; }
   .panel {
      border: 1px solid #00ffcc;
      border-radius: 8px;
      padding: 15px;
      margin-bottom: 15px;
      background: #131722;
      box-shadow: 0 0 10px rgba(0, 255, 204, 0.1);
    }
   .hidden { display: none!important; }
   .room-setup {
      display: flex;
      gap: 10px;
      align-items: center;
      margin-top: 10px;
      flex-wrap: wrap;
    }
    input[type="text"] {
      background: #1a202c;
      border: 1px solid #00ffcc;
      color: #fff;
      padding: 8px 12px;
      border-radius: 4px;
      font-size: 16px;
      letter-spacing: 2px;
      text-align: center;
    }
    button {
      background: #00ffcc;
      color: #0d0f17;
      border: none;
      padding: 8px 16px;
      font-weight: bold;
      border-radius: 4px;
      cursor: pointer;
      transition: all 0.2s;
    }
    button:hover {
      background: #00cca3;
      box-shadow: 0 0 8px #00ffcc;
    }
    button:disabled {
      background: #4a5568;
      color: #718096;
      cursor: not-allowed;
      box-shadow: none;
    }
   .battlefield {
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
   .player-board {
      border: 1px solid #2d3748;
      border-radius: 6px;
      padding: 10px;
      background: #1a202c;
    }
   .stats-bar {
      display: flex;
      gap: 20px;
      font-size: 16px;
      font-weight: bold;
      flex-wrap: wrap;
    }
   .stat-item {
      background: #0d0f17;
      padding: 4px 10px;
      border-radius: 4px;
      border: 1px solid #4a5568;
    }
   .hand-container {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 10px;
      min-height: 120px;
    }
   .card {
      width: 120px;
      height: 160px;
      border: 2px solid #00ffcc;
      border-radius: 8px;
      padding: 8px;
      background: #111827;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      cursor: pointer;
      user-select: none;
      transition: transform 0.2s, box-shadow 0.2s;
    }
   .card:hover:not(.disabled) {
      transform: translateY(-5px);
      box-shadow: 0 5px 15px rgba(0, 255, 204, 0.4);
    }
   .card.disabled {
      border-color: #4a5568;
      color: #4a5568;
      opacity: 0.5;
      cursor: not-allowed;
    }
   .card.discard-mode {
      border-color: #ff0055!important;
      box-shadow: 0 0 10px rgba(255, 0, 85, 0.5);
    }
   .card.discard-mode:hover {
      transform: scale(1.05);
      background: #32111e;
    }
   .card-title { font-weight: bold; font-size: 14px; text-align: center; }
   .card-type { font-size: 10px; color: #a0aec0; text-align: center; }
   .card-cost { font-size: 12px; font-weight: bold; color: #ff007f; }
   .top-bar { display: flex; justify-content: space-between; align-items: center; }
  </style>
</head>
<body>
  <h1>Cyberpunk 駭客攻防戰 (網路連線版)</h1>
  <div id="room-panel" class="panel">
    <h2>網路對戰大廳</h2>
    <p>點擊創建房間後會取得 4 位數代碼，將代碼告訴對手即可加入：</p>
    <div class="room-setup">
      <button onclick="createRoom()">創建新房間</button>
      <span>或</span>
      <input type="text" id="room-input" placeholder="輸入 4 位數房號" maxlength="4" style="width: 150px;">
      <button onclick="joinRoom()">加入房間</button>
    </div>
    <p id="room-status" style="color: #ff007f; margin-top: 10px;"></p>
  </div>

  <div id="game-panel" class="panel hidden">
    <div class="top-bar">
      <div>
        <span>房間代碼：<b id="current-room-id" style="color:#ff007f;">-</b></span> |
        <span>你的身份：<b>Player <span id="my-player-id">-</span></b></span>
      </div>
      <button id="bgm-btn" onclick="toggleBGM()">🔊 背景音樂：播放中</button>
    </div>
    <hr style="border-color: #2d3748; margin: 10px 0;">
    <h3>
      環境狀態：<span id="day-night-state" style="color:#ff007f;">Daytime (白天)</span> |
      回合進行：Player <span id="turn-player-id">0</span>
    </h3>
    <div class="battlefield">
      <div class="player-board">
        <h4>對手 (Player <span id="opp-id"></span>)</h4>
        <div class="stats-bar">
          <div class="stat-item">HP: <span id="opp-hp">10</span>/10</div>
          <div class="stat-item">防火牆: <span id="opp-fw">0</span>/3</div>
          <div class="stat-item">電力: <span id="opp-power">7</span>/7</div>
          <div class="stat-item">手牌數: <span id="opp-hand-count">0</span></div>
          <div class="stat-item">牌庫: <span id="opp-deck-count">0</span>張</div>
        </div>
      </div>
      <div class="player-board">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <h4>你的區域 (Player <span id="my-id"></span>)</h4>
          <button id="end-turn-btn" onclick="tryEndTurn()">結束回合</button>
        </div>
        <div class="stats-bar" style="margin-bottom: 10px;">
          <div class="stat-item">HP: <span id="my-hp">10</span>/10</div>
          <div class="stat-item">防火牆: <span id="my-fw">0</span>/3</div>
          <div class="stat-item">電力: <span id="my-power">7</span>/7</div>
          <div class="stat-item">牌庫: <span id="my-deck-count">0</span>張</div>
        </div>
        <div>
          <span>你的手牌（手牌上限等於當前 HP）：<span id="hand-limit-hint" style="color:#ff0055; font-size: 13px; font-weight: bold;"></span></span>
          <div id="hand-cards" class="hand-container"></div>
        </div>
      </div>
    </div>
  </div>

  <script>
    let peer = null;
    let conn = null;
    let roomId = null;
    let myIndex = 0;
    let isMyTurn = false;
    let isDiscardingMode = false;

    const PEER_PREFIX = "cyberpunk_card_game_";

    // ----- Web Audio API 聲音引擎 -----
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    let isBgmPlaying = true;
    let bgmInterval = null;
    let noteIndex = 0;

    const NOTES = {
      A1: 55.00, C2: 65.41, D2: 73.42, E2: 82.41, G2: 97.99,
      A2: 110.00, C3: 130.81, D3: 146.83, E3: 164.81, G3: 196.00,
      A3: 220.00, C4: 261.63, D4: 293.66, E4: 329.63, G4: 392.00
    };

    const bassSequence = [
      NOTES.A1, NOTES.A1, NOTES.C2, NOTES.A1, NOTES.D2, NOTES.A1, NOTES.C2, NOTES.G2,
      NOTES.A1, NOTES.A1, NOTES.C2, NOTES.A1, NOTES.E2, NOTES.D2, NOTES.C2, NOTES.A1
    ];
    const synthSequence = [
      NOTES.A3, 0, NOTES.C4, 0, NOTES.D4, NOTES.E4, 0, NOTES.G4,
      NOTES.E4, 0, NOTES.D4, 0, NOTES.C4, NOTES.A3, NOTES.G3, 0
    ];

    function playBgmStep() {
      if (!isBgmPlaying) return;
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const now = audioCtx.currentTime;

      const bassFreq = bassSequence[noteIndex % bassSequence.length];
      if (bassFreq > 0) {
        const bassOsc = audioCtx.createOscillator();
        const bassGain = audioCtx.createGain();
        bassOsc.type = 'sawtooth';
        bassOsc.frequency.setValueAtTime(bassFreq, now);
        bassGain.gain.setValueAtTime(0.08, now);
        bassGain.gain.exponentialRampToValueAtTime(0.001, now + 0.15);
        bassOsc.connect(bassGain);
        bassGain.connect(audioCtx.destination);
        bassOsc.start(now);
        bassOsc.stop(now + 0.15);
      }

      const synthFreq = synthSequence[noteIndex % synthSequence.length];
      if (synthFreq > 0) {
        const synthOsc = audioCtx.createOscillator();
        const synthGain = audioCtx.createGain();
        synthOsc.type = 'square';
        synthOsc.frequency.setValueAtTime(synthFreq, now);
        synthGain.gain.setValueAtTime(0.03, now);
        synthGain.gain.exponentialRampToValueAtTime(0.001, now + 0.2);
        synthOsc.connect(synthGain);
        synthGain.connect(audioCtx.destination);
        synthOsc.start(now);
        synthOsc.stop(now + 0.2);
      }
      noteIndex++;
    }

    function startBGM() {
      if (!bgmInterval) {
        isBgmPlaying = true;
        bgmInterval = setInterval(playBgmStep, 150);
      }
    }

    function toggleBGM() {
      const btn = document.getElementById('bgm-btn');
      if (audioCtx.state === 'suspended') audioCtx.resume();
      isBgmPlaying = !isBgmPlaying;
      btn.innerText = isBgmPlaying ? "🔊 背景音樂：播放中" : "🔇 背景音樂：靜音";
    }

    function playSfx(type) {
      if (audioCtx.state === 'suspended') audioCtx.resume();
      const now = audioCtx.currentTime;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      osc.connect(gain);
      gain.connect(audioCtx.destination);

      if (type === 'play_card') {
        osc.type = 'sine';
        osc.frequency.setValueAtTime(400, now);
        osc.frequency.exponentialRampToValueAtTime(900, now + 0.1);
        gain.gain.setValueAtTime(0.2, now);
        gain.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
        osc.start(now);
        osc.stop(now + 0.1);
      } else if (type === 'attack') {
        osc.type = 'sawtooth';
        osc.frequency.setValueAtTime(180, now);
        osc.frequency.exponentialRampToValueAtTime(30, now + 0.2);
        gain.gain.setValueAtTime(0.3, now);
        gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
        osc.start(now);
        osc.stop(now + 0.2);
      } else if (type === 'end_turn') {
        osc.type = 'triangle';
        osc.frequency.setValueAtTime(523.25, now);
        osc.frequency.setValueAtTime(659.25, now + 0.08);
        gain.gain.setValueAtTime(0.15, now);
        gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);
        osc.start(now);
        osc.stop(now + 0.2);
      }
    }

    // ----- 遊戲邏輯與 75 張卡牌定義（微調後版本） -----
    const CARD_TYPES = {
      firewall: { name: "防火牆", cost: 0, type: "defense", desc: "獲得 1 層防火牆(上限3)" },
      repair: { name: "維修", cost: 1, type: "defense", desc: "回復 1 HP (殘血 HP≤1 時可在對手回合發動)" },
      emergency_power: { name: "搶修電力", cost: 0, type: "skill", desc: "扣 1 HP，恢復 3 電力" },
      invasion: { name: "入侵", cost: 1, type: "attack", desc: "造成1傷害(白天2傷害)" },
      destroy_fw: { name: "防火牆破壞", cost: 2, type: "attack", desc: "摧毀最多 2 層防火牆" },
      delete_data: { name: "刪除資料", cost: 1, type: "attack", desc: "迫使對手隨機棄置 1 張手牌" },
      steal_data: { name: "資料竊取", cost: 2, type: "attack", desc: "隨機偷取對手 1 張手牌" },
      energy_surge: { name: "導能電擊", cost: 0, type: "attack", desc: "消耗所有剩餘電力，每2電力造成1傷害" },
      data_rebuild: { name: "數據重組", cost: 2, type: "skill", desc: "抽取 2 張卡牌" },
      system_overload: { name: "系統超載", cost: 3, type: "skill", desc: "強制切換晝夜狀態" }
    };

    let gameState = {
      turn: 0,
      roundCount: 0,
      isDaytime: true,
      players: [
        { hp: 10, firewall: 0, power: 7, hand: [], deck: [], discard: [], invasionUsed: false },
        { hp: 10, firewall: 0, power: 7, hand: [], deck: [], discard: [], invasionUsed: false }
      ]
    };

    function build75Deck() {
      let deck = [];
      const add = (key, count) => { for(let i=0; i<count; i++) deck.push(key); };
      add('firewall', 16);          // +2
      add('repair', 7);             // -4
      add('emergency_power', 6);
      add('invasion', 18);          // +4
      add('destroy_fw', 5);         // -3
      add('delete_data', 6);
      add('steal_data', 4);
      add('energy_surge', 4);
      add('data_rebuild', 5);
      add('system_overload', 4);    // +1
      shuffleArray(deck);
      return deck;
    }

    function shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }

    function drawCardForPlayer(p) {
      if (p.deck.length === 0) {
        if (p.discard.length === 0) return false;
        p.deck = [...p.discard];
        p.discard = [];
        shuffleArray(p.deck);
      }
      p.hand.push(p.deck.pop());
      return true;
    }

    function createRoom() {
      myIndex = 0;
      const shortCode = Math.floor(1000 + Math.random() * 9000).toString();
      roomId = shortCode;
      
      peer = new Peer(PEER_PREFIX + shortCode);
      
      peer.on('open', () => {
        document.getElementById('room-status').innerText = `房間已建立！房號為：${shortCode}\n請將這 4 位數告訴對手，等待對方加入...`;
      });

      peer.on('error', (err) => {
        if (err.type === 'unavailable-id') {
          createRoom();
        } else {
          alert("連線發生錯誤：" + err.type);
        }
      });

      peer.on('connection', (connection) => {
        conn = connection;
        setupConnection();
        startGameUI();
        
        conn.on('open', () => {
          initGameState();
        });
      });
    }

    function joinRoom() {
      const shortCode = document.getElementById('room-input').value.trim();
      if (!shortCode || shortCode.length !== 4) { alert("請輸入正確的 4 位數房號！"); return; }
      
      myIndex = 1;
      roomId = shortCode;
      peer = new Peer();

      peer.on('open', () => {
        conn = peer.connect(PEER_PREFIX + shortCode);
        setupConnection();
        startGameUI();
      });

      peer.on('error', (err) => {
        alert("無法連線至該房間，請確認房號是否正確！");
      });
    }

    function setupConnection() {
      conn.on('data', (data) => {
        handleServerAction(data.action, data.payload, data.sender);
      });
      conn.on('close', () => {
        alert("對手已斷開連線！");
        location.reload();
      });
    }

    function broadcastAction(action, payload = {}) {
      if (conn && conn.open) {
        conn.send({ action, payload, sender: myIndex });
      }
    }

    function sendAction(action, payload = {}) {
      handleServerAction(action, payload, myIndex);
      broadcastAction(action, payload);
    }

    function startGameUI() {
      document.getElementById('room-panel').classList.add('hidden');
      document.getElementById('game-panel').classList.remove('hidden');
      document.getElementById('current-room-id').innerText = roomId;
      document.getElementById('my-player-id').innerText = myIndex;
      document.getElementById('my-id').innerText = myIndex;
      document.getElementById('opp-id').innerText = myIndex === 0 ? 1 : 0;
      startBGM();
    }

    function initGameState() {
      gameState.players[0].deck = build75Deck();
      gameState.players[1].deck = build75Deck();
      gameState.players[0].discard = [];
      gameState.players[1].discard = [];
      for (let i = 0; i < 4; i++) {
        drawCardForPlayer(gameState.players[0]);
        drawCardForPlayer(gameState.players[1]);
      }
      updateUI();
      broadcastAction('SYNC_ALL', gameState);
    }

    function checkWin() {
      for (let i = 0; i < 2; i++) {
        if (gameState.players[i].hp <= 0) {
          const winner = i === 0 ? 1 : 0;
          setTimeout(() => {
            alert(`遊戲結束！ Player ${winner} 獲勝！`);
            location.reload();
          }, 300);
          return true;
        }
      }
      return false;
    }

    function tryEndTurn() {
      const me = gameState.players[myIndex];
      if (me.hand.length > me.hp) {
        isDiscardingMode = true;
        updateUI();
      } else {
        isDiscardingMode = false;
        playSfx('end_turn');
        sendAction('END_TURN');
      }
    }

    function handleServerAction(action, payload, sender) {
      if (action === 'SYNC_ALL') {
        gameState = payload;
      } else if (action === 'PLAY_CARD') {
        executeCardEffect(sender, payload.cardKey, payload.cardIdx);
      } else if (action === 'DISCARD_CARD') {
        const p = gameState.players[sender];
        const discarded = p.hand.splice(payload.cardIdx, 1)[0];
        if (discarded) p.discard.push(discarded);
      } else if (action === 'END_TURN') {
        gameState.turn = gameState.turn === 0 ? 1 : 0;
        
        if (gameState.turn === 0) {
          gameState.roundCount++;
          gameState.isDaytime = !gameState.isDaytime;
        }

        const nextPlayer = gameState.players[gameState.turn];
        nextPlayer.power = Math.min(7, nextPlayer.power + 4);
        nextPlayer.invasionUsed = false;
        for (let i = 0; i < 2; i++) {
          drawCardForPlayer(nextPlayer);
        }
      }
      if (gameState.players[myIndex].hand.length <= gameState.players[myIndex].hp) {
        isDiscardingMode = false;
      }
      updateUI();
      checkWin();
    }

    function executeCardEffect(playerIdx, cardKey, cardIdx) {
      const p = gameState.players[playerIdx];
      const opp = gameState.players[playerIdx === 0 ? 1 : 0];
      const card = CARD_TYPES[cardKey];
      if (!card) return;

      let actualCost = card.cost;
      if (card.type === 'skill' && gameState.isDaytime && cardKey !== 'system_overload') {
        actualCost *= 2;
      }

      let energySpent = p.power;
      let calculatedDmg = 0;
      if (cardKey === 'energy_surge') {
        energySpent = p.power;
        calculatedDmg = Math.floor(energySpent / 2);
        p.power = 0;
      } else {
        p.power -= actualCost;
      }

      const playedCard = p.hand.splice(cardIdx, 1)[0];
      if (playedCard) p.discard.push(playedCard);
      
      playSfx('play_card');

      if (cardKey === 'firewall') {
        p.firewall = Math.min(3, p.firewall + 1);
      } else if (cardKey === 'repair') {
        p.hp = Math.min(10, p.hp + 1);
      } else if (cardKey === 'emergency_power') {
        p.hp = Math.max(0, p.hp - 1);
        p.power = Math.min(7, p.power + 3);
      } else if (cardKey === 'invasion') {
        p.invasionUsed = true;
        playSfx('attack');
        let dmg = gameState.isDaytime ? 2 : 1;
        if (opp.firewall >= dmg) {
          opp.firewall -= dmg;
        } else {
          dmg -= opp.firewall;
          opp.firewall = 0;
          opp.hp = Math.max(0, opp.hp - dmg);
        }
      } else if (cardKey === 'destroy_fw') {
        playSfx('attack');
        opp.firewall = Math.max(0, opp.firewall - 2);
      } else if (cardKey === 'delete_data') {
        playSfx('attack');
        if (opp.hand.length > 0) {
          const randIdx = Math.floor(Math.random() * opp.hand.length);
          const removed = opp.hand.splice(randIdx, 1)[0];
          opp.discard.push(removed);
        }
      } else if (cardKey === 'steal_data') {
        playSfx('attack');
        if (opp.hand.length > 0) {
          const randIdx = Math.floor(Math.random() * opp.hand.length);
          const stolen = opp.hand.splice(randIdx, 1)[0];
          p.hand.push(stolen);
        }
      } else if (cardKey === 'energy_surge') {
        playSfx('attack');
        if (calculatedDmg > 0) {
          if (opp.firewall >= calculatedDmg) {
            opp.firewall -= calculatedDmg;
          } else {
            calculatedDmg -= opp.firewall;
            opp.firewall = 0;
            opp.hp = Math.max(0, opp.hp - calculatedDmg);
          }
        }
      } else if (cardKey === 'data_rebuild') {
        for (let i = 0; i < 2; i++) {
          drawCardForPlayer(p);
        }
      } else if (cardKey === 'system_overload') {
        gameState.isDaytime = !gameState.isDaytime;
      }
    }

    function updateUI() {
      isMyTurn = gameState.turn === myIndex;
      const me = gameState.players[myIndex];
      const opp = gameState.players[myIndex === 0 ? 1 : 0];

      document.getElementById('turn-player-id').innerText = gameState.turn;
      document.getElementById('day-night-state').innerText = gameState.isDaytime ? "Daytime (白天)" : "Night time (黑夜)";
      document.getElementById('my-hp').innerText = me.hp;
      document.getElementById('my-fw').innerText = me.firewall;
      document.getElementById('my-power').innerText = me.power;
      document.getElementById('my-deck-count').innerText = me.deck.length;
      document.getElementById('opp-hp').innerText = opp.hp;

      if (gameState.isDaytime) {
        document.getElementById('opp-fw').innerText = opp.firewall;
        document.getElementById('opp-power').innerText = opp.power;
      } else {
        document.getElementById('opp-fw').innerText = "?";
        document.getElementById('opp-power').innerText = "?";
      }

      document.getElementById('opp-hand-count').innerText = opp.hand.length;
      document.getElementById('opp-deck-count').innerText = opp.deck.length;

      const endBtn = document.getElementById('end-turn-btn');
      endBtn.disabled = !isMyTurn;

      const hintEl = document.getElementById('hand-limit-hint');
      if (isDiscardingMode) {
        const overflow = me.hand.length - me.hp;
        hintEl.innerText = `⚠️ 手牌數 (${me.hand.length}) 超過當前血量上限 (${me.hp})！請選擇 ${overflow} 張卡牌棄掉。`;
        endBtn.disabled = true;
      } else if (me.hand.length > me.hp) {
        hintEl.innerText = `(提示：手牌超過上限，回合結束需選擇棄牌)`;
      } else {
        hintEl.innerText = ``;
      }

      const handContainer = document.getElementById('hand-cards');
      handContainer.innerHTML = '';

      me.hand.forEach((cardKey, idx) => {
        const card = CARD_TYPES[cardKey];
        const cardEl = document.createElement('div');
        let actualCost = card.cost;
        if (card.type === 'skill' && gameState.isDaytime && cardKey !== 'system_overload') {
          actualCost *= 2;
        }
        if (cardKey === 'energy_surge') {
          actualCost = me.power;
        }

        cardEl.className = 'card';
        cardEl.innerHTML = `
          <div class="card-title">${card.name}</div>
          <div class="card-type">${card.type.toUpperCase()}</div>
          <div style="font-size:11px; text-align:center;">${card.desc}</div>
          <div class="card-cost">${isDiscardingMode ? '點擊棄牌' : (cardKey === 'energy_surge' ? '電力: 全部(' + me.power + ')' : '電力: ' + actualCost)}</div>
        `;

        if (isDiscardingMode) {
          cardEl.classList.add('discard-mode');
          cardEl.onclick = () => sendAction('DISCARD_CARD', { cardIdx: idx });
        } else {
          let canPlay = true;
          if (!isMyTurn) {
            canPlay = (cardKey === 'repair' && me.hp <= 1);
          }
          if (!gameState.isDaytime && card.type === 'skill' && cardKey !== 'system_overload') {
            canPlay = false;
          }
          if (cardKey === 'invasion' && me.invasionUsed) canPlay = false;
          
          if ((cardKey === 'delete_data' || cardKey === 'steal_data') && opp.hand.length === 0) {
            canPlay = false;
          }

          if (cardKey === 'energy_surge' && me.power <= 0) canPlay = false;
          else if (cardKey !== 'energy_surge' && me.power < actualCost) canPlay = false;
          if (cardKey === 'emergency_power' && me.hp <= 1) canPlay = false;

          if (canPlay) {
            cardEl.onclick = () => sendAction('PLAY_CARD', { cardKey, cardIdx: idx });
          } else {
            cardEl.classList.add('disabled');
          }
        }
        handContainer.appendChild(cardEl);
      });
    }
  </script>
</body>
</html>
