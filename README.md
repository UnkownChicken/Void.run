<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Void.Run - Solo .IO Game</title>
<style>
  html, body {
    margin: 0; padding: 0; overflow: hidden;
    background-color: #0b0c10;
    color: #fff;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    user-select: none;
  }
  canvas {
    display: block;
    background: radial-gradient(#111, #000);
  }
  #score, #coins {
    position: absolute;
    top: 10px; left: 10px;
    font-size: 18px;
    background: rgba(0,0,0,0.6);
    padding: 10px 15px;
    border-radius: 8px;
  }
  #coins { top: 50px; }
  #mainMenu, #howToPlay, #gameOver, #shop {
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: rgba(0,0,0,0.85);
    padding: 30px 40px;
    border-radius: 15px;
    text-align: center;
    display: none;
    width: 90%;
    max-width: 400px;
  }
  #mainMenu h1, #howToPlay h1, #gameOver h1, #shop h1 {
    font-size: 3rem;
    margin-bottom: 20px;
  }
  button {
    font-size: 1.3rem;
    background: #1f1f1f;
    color: #fff;
    padding: 12px 25px;
    border: none;
    cursor: pointer;
    border-radius: 12px;
    margin: 8px;
  }
  button:disabled {
    background: #555;
    cursor: default;
  }
  #touchControls {
    position: absolute;
    bottom: 30px;
    width: 100%;
    text-align: center;
    display: none;
  }
  .touchBtn {
    font-size: 1.6rem;
    padding: 16px 22px;
    margin: 6px;
    border-radius: 12px;
  }
  #shopSkins {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    gap: 20px;
    margin-top: 20px;
  }
  .skinCard {
    width: 80px; height: 80px;
    border-radius: 50%;
    cursor: pointer;
    border: 3px solid transparent;
    position: relative;
    transition: transform 0.3s ease;
  }
  .skinCard:hover {
    transform: scale(1.1);
  }
  .skinCard.selected {
    border-color: #00ffff;
  }
  .lockedOverlay {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: rgba(0,0,0,0.7);
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #f00;
    font-weight: bold;
    font-size: 1.2rem;
  }
  /* Joystick styles */
  #joystickContainer {
    position: absolute;
    bottom: 40px;
    left: 40px;
    width: 120px;
    height: 120px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 50%;
    touch-action: none;
    display: none;
  }
  #joystickBase {
    position: absolute;
    width: 100%;
    height: 100%;
    background: rgba(255,255,255,0.15);
    border-radius: 50%;
  }
  #joystickStick {
    position: absolute;
    width: 60px;
    height: 60px;
    background: #00ffff;
    border-radius: 50%;
    top: 30px;
    left: 30px;
    transition: top 0.1s, left 0.1s;
  }
  #joystickToggle {
    position: absolute;
    bottom: 10px;
    left: 10px;
    background: #222;
    border-radius: 10px;
    padding: 8px 12px;
    font-size: 14px;
    cursor: pointer;
    user-select: none;
    display: none;
  }
</style>
</head>
<body>

<canvas id="game"></canvas>
<div id="score">Energy: 100</div>
<div id="coins">Coins: 0</div>

<!-- Main Menu -->
<div id="mainMenu">
  <h1>Void.Run</h1>
  <button onclick="showHowToPlay()">How to Play</button>
  <button onclick="showShop()">Shop</button>
  <button onclick="startGame()">Start Game</button>
</div>

<!-- How to Play -->
<div id="howToPlay">
  <h1>How to Play</h1>
  <p>Move your orb using arrow keys or on-screen joystick/buttons.</p>
  <p>Collect <span style="color:#ffff00;">light shards</span> to restore energy.</p>
  <p>Collect <span style="color:#ffd700;">coins</span> to unlock new orb skins in the shop.</p>
  <p>Avoid <span style="color:#800080;">purple mines</span> and <span style="color:#ff0000;">red enemies</span>.</p>
  <button onclick="hideHowToPlay()">Back</button>
</div>

<!-- Shop -->
<div id="shop">
  <h1>Shop</h1>
  <div id="shopSkins"></div>
  <button onclick="hideShop()">Back</button>
</div>

<!-- Game Over -->
<div id="gameOver">
  <h1>Game Over</h1>
  <button onclick="restartGame()">Restart</button>
  <button onclick="showMainMenu()">Main Menu</button>
</div>

<!-- Touch Controls Buttons (Fallback) -->
<div id="touchControls">
  <button class="touchBtn" onclick="moveOrb('ArrowUp')">⬆️</button><br />
  <button class="touchBtn" onclick="moveOrb('ArrowLeft')">⬅️</button>
  <button class="touchBtn" onclick="moveOrb('ArrowDown')">⬇️</button>
  <button class="touchBtn" onclick="moveOrb('ArrowRight')">➡️</button>
</div>

<!-- Joystick -->
<div id="joystickToggle">Toggle Joystick</div>
<div id="joystickContainer">
  <div id="joystickBase"></div>
  <div id="joystickStick"></div>
</div>

<script>
  // Canvas setup
  const canvas = document.getElementById("game");
  const ctx = canvas.getContext("2d");
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  // Orb player data
  let orb = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    radius: 12,
    speed: 3,
    energy: 100,
    coins: 0,
    skin: "default"
  };

  // Skins available in the shop
  const skins = [
    { id: "default", color: "#00ffff", cost: 0, unlocked: true, effect: () => drawGlow("#00ffff") },
    { id: "pink", color: "#ff69b4", cost: 15, unlocked: false, effect: () => drawGlow("#ff69b4") },
    { id: "green", color: "#00ff00", cost: 20, unlocked: false, effect: () => drawGlow("#00ff00") },
    { id: "gold", color: "#ffd700", cost: 30, unlocked: false, effect: () => drawGlow("#ffd700") },
    { id: "purple", color: "#800080", cost: 40, unlocked: false, effect: () => drawGlow("#800080") }
  ];

  // DOM elements references
  const scoreDiv = document.getElementById("score");
  const coinsDiv = document.getElementById("coins");
  const mainMenu = document.getElementById("mainMenu");
  const howToPlay = document.getElementById("howToPlay");
  const gameOverDiv = document.getElementById("gameOver");
  const shopDiv = document.getElementById("shop");
  const shopSkinsDiv = document.getElementById("shopSkins");
  const touchControls = document.getElementById("touchControls");
  const joystickToggle = document.getElementById("joystickToggle");
  const joystickContainer = document.getElementById("joystickContainer");
  const joystickStick = document.getElementById("joystickStick");

  // Local storage keys for persistence
  const STORAGE_KEY_COINS = "voidrun_coins";
  const STORAGE_KEY_UNLOCKED = "voidrun_unlocked_skins";
  const STORAGE_KEY_SELECTED = "voidrun_selected_skin";
  const STORAGE_KEY_JOYSTICK = "voidrun_joystick_enabled";

  // Game state variables
  let keys = {};
  let lightShards = [], coinsArr = [], enemies = [], mines = [];
  let frame = 0;
  let isGameOver = false;
  let inGame = false;

  // Joystick state variables
  let joystickEnabled = localStorage.getItem(STORAGE_KEY_JOYSTICK) === "true";
  let joystickActive = false;
  let joystickRadius = 60;
  let stickRadius = 30;
  let joystickVector = { x: 0, y: 0 };

  // Load saved game data
  function loadGameData() {
    orb.coins = parseInt(localStorage.getItem(STORAGE_KEY_COINS)) || 0;
    let unlocked = JSON.parse(localStorage.getItem(STORAGE_KEY_UNLOCKED));
    if (unlocked && Array.isArray(unlocked)) {
      for (let skin of skins) {
        if (unlocked.includes(skin.id)) skin.unlocked = true;
      }
    }
    const selected = localStorage.getItem(STORAGE_KEY_SELECTED);
    if (selected && skins.some(s => s.id === selected)) {
      orb.skin = selected;
    }
  }

  // Save game data
  function saveGameData() {
    localStorage.setItem(STORAGE_KEY_COINS, orb.coins);
    const unlockedSkins = skins.filter(s => s.unlocked).map(s => s.id);
    localStorage.setItem(STORAGE_KEY_UNLOCKED, JSON.stringify(unlockedSkins));
    localStorage.setItem(STORAGE_KEY_SELECTED, orb.skin);
  }

  // Save joystick toggle state
  function saveJoystickSetting() {
    localStorage.setItem(STORAGE_KEY_JOYSTICK, joystickEnabled);
  }

  // Draw glow effect around orb based on skin color
  function drawGlow(color) {
    let gradient = ctx.createRadialGradient(orb.x, orb.y, orb.radius / 2, orb.x, orb.y, orb.radius * 3);
    gradient.addColorStop(0, color);
    gradient.addColorStop(1, "transparent");
    ctx.shadowColor = color;
    ctx.shadowBlur = 15;
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.arc(orb.x, orb.y, orb.radius, 0, Math.PI * 2);
    ctx.fill();
    ctx.shadowBlur = 0;
  }

  // Draw the orb with current skin color and glow
  function drawOrb() {
    ctx.shadowColor = "transparent";
    ctx.beginPath();
    ctx.arc(orb.x, orb.y, orb.radius, 0, Math.PI * 2);
    const skinObj = skins.find(s => s.id === orb.skin);
    ctx.fillStyle = skinObj ? skinObj.color : "#00ffff";
    ctx.fill();
    if (skinObj && skinObj.effect) skinObj.effect();
  }

  // Draw a colored circle for objects like light shards, coins, enemies, mines
  function drawCircle(obj, color) {
    ctx.beginPath();
    ctx.arc(obj.x, obj.y, obj.radius || 5, 0, Math.PI * 2);
    ctx.fillStyle = color;
    ctx.fill();
  }

  // Spawn game objects periodically
  function spawnObjects() {
    if (frame % 30 === 0 && lightShards.length < 50) {
      lightShards.push({ x: Math.random() * canvas.width, y: Math.random() * canvas.height, radius: 6 });
    }
    if (frame % 90 === 0 && coinsArr.length < 15) {
      coinsArr.push({ x: Math.random() * canvas.width, y: Math.random() * canvas.height, radius: 6 });
    }
    if (frame % 120 === 0 && enemies.length < 5) {
      enemies.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        radius: 10,
        dx: Math.random() * 2 - 1,
        dy: Math.random() * 2 - 1
      });
    }
    if (frame % 150 === 0 && mines.length < 5) {
      mines.push({ x: Math.random() * canvas.width, y: Math.random() * canvas.height, radius: 8 });
    }
  }

  // Collision detection helper
  function collide(a, b) {
    let dx = a.x - b.x, dy = a.y - b.y;
    return Math.sqrt(dx*dx + dy*dy) < (a.radius + (b.radius || 5));
  }

  // Check collisions between orb and array of objects, run effect if collided
  function checkCollision(array, effect) {
    for (let i = array.length - 1; i >= 0; i--) {
      if (collide(orb, array[i])) {
        effect(array[i], i);
        return true;
      }
    }
    return false;
  }

  // Update orb position based on keys & joystick input
  function update() {
    if (joystickEnabled && joystickVector.x !== 0 || joystickVector.y !== 0) {
      orb.x += joystickVector.x * orb.speed;
      orb.y += joystickVector.y * orb.speed;
    } else {
      if (keys["ArrowUp"]) orb.y -= orb.speed;
      if (keys["ArrowDown"]) orb.y += orb.speed;
      if (keys["ArrowLeft"]) orb.x -= orb.speed;
      if (keys["ArrowRight"]) orb.x += orb.speed;
    }

    // Keep orb inside canvas bounds
    orb.x = Math.min(Math.max(orb.radius, orb.x), canvas.width - orb.radius);
    orb.y = Math.min(Math.max(orb.radius, orb.y), canvas.height - orb.radius);

    orb.energy -= 0.05;
    if (orb.energy <= 0) return endGame();

    // Check collisions with light shards (restore energy)
    checkCollision(lightShards, (item, idx) => {
      orb.energy = Math.min(100, orb.energy + 10);
      lightShards.splice(idx,1);
    });

    // Check collisions with coins (increment coins)
    checkCollision(coinsArr, (item, idx) => {
      orb.coins++;
      coinsArr.splice(idx,1);
      saveGameData();
      coinsDiv.textContent = `Coins: ${orb.coins}`;
    });

    // Check collisions with enemies and mines (game over)
    if (checkCollision(enemies, () => endGame())) return;
    if (checkCollision(mines, () => endGame())) return;

    // Move enemies and bounce off edges
    enemies.forEach(e => {
      e.x += e.dx;
      e.y += e.dy;
      if (e.x < e.radius || e.x > canvas.width - e.radius) e.dx *= -1;
      if (e.y < e.radius || e.y > canvas.height - e.radius) e.dy *= -1;
    });

    scoreDiv.textContent = `Energy: ${Math.round(orb.energy)}`;
  }

  // Draw game frame
  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawOrb();
    lightShards.forEach(s => drawCircle(s, "#ffff00"));
    coinsArr.forEach(c => drawCircle(c, "#ffd700"));
    mines.forEach(m => drawCircle(m, "#800080"));
    enemies.forEach(e => drawCircle(e, "#ff0000"));
  }

  // Main game loop
  function gameLoop() {
    if (!isGameOver && inGame) {
      requestAnimationFrame(gameLoop);
      spawnObjects();
      update();
      draw();
      frame++;
    }
  }

  // Touch button fallback controls
  function moveOrb(key) {
    keys[key] = true;
    setTimeout(() => { keys[key] = false; }, 100);
  }

  // Game control functions
  function startGame() {
    orb.x = canvas.width / 2;
    orb.y = canvas.height / 2;
    orb.energy = 100;
    frame = 0;
    lightShards = [];
    coinsArr = [];
    enemies = [];
    mines = [];
    isGameOver = false;
    inGame = true;
    mainMenu.style.display = "none";
    howToPlay.style.display = "none";
    gameOverDiv.style.display = "none";
    shopDiv.style.display = "none";
    loadGameData();
    coinsDiv.textContent = `Coins: ${orb.coins}`;
    applySkin(orb.skin);

    // Show joystick toggle & controls on game start
    joystickToggle.style.display = "block";
    updateJoystickVisibility();

    gameLoop();
  }

  function endGame() {
    isGameOver = true;
    inGame = false;
    gameOverDiv.style.display = "block";

    // Hide joystick and touch buttons on game over
    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  function restartGame() {
    startGame();
  }

  function showMainMenu() {
    gameOverDiv.style.display = "none";
    howToPlay.style.display = "none";
    shopDiv.style.display = "none";
    mainMenu.style.display = "block";

    // Hide joystick and touch buttons on main menu
    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  function showHowToPlay() {
    mainMenu.style.display = "none";
    howToPlay.style.display = "block";

    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  function hideHowToPlay() {
    howToPlay.style.display = "none";
    mainMenu.style.display = "block";

    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  function showShop() {
    mainMenu.style.display = "none";
    shopDiv.style.display = "block";
    renderShop();

    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  function hideShop() {
    shopDiv.style.display = "none";
    mainMenu.style.display = "block";

    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
  }

  // Shop rendering
  function renderShop() {
    shopSkinsDiv.innerHTML = "";
    skins.forEach(skin => {
      const skinDiv = document.createElement("div");
      skinDiv.className = "skinCard";
      skinDiv.style.backgroundColor = skin.color;
      if (skin.id === orb.skin) skinDiv.classList.add("selected");
      skinDiv.style.boxShadow = `0 0 10px 3px ${skin.color}`;
      if (!skin.unlocked) {
        const overlay = document.createElement("div");
        overlay.className = "lockedOverlay";
        overlay.textContent = `${skin.cost} 💰`;
        skinDiv.appendChild(overlay);
      }
      skinDiv.onclick = () => {
        if (skin.unlocked) {
          orb.skin = skin.id;
          applySkin(skin.id);
          saveGameData();
          renderShop();
        } else if (orb.coins >= skin.cost) {
          orb.coins -= skin.cost;
          skin.unlocked = true;
          orb.skin = skin.id;
          applySkin(skin.id);
          saveGameData();
          coinsDiv.textContent = `Coins: ${orb.coins}`;
          renderShop();
          alert(`Unlocked ${skin.id} skin!`);
        } else {
          alert(`Not enough coins! You need ${skin.cost} coins.`);
        }
      };
      shopSkinsDiv.appendChild(skinDiv);
    });
  }

  function applySkin(skinId) {
    const skin = skins.find(s => s.id === skinId);
    if (skin) orb.skin = skinId;
  }

  // Keyboard event listeners
  window.addEventListener("keydown", e => { keys[e.key] = true; });
  window.addEventListener("keyup", e => { keys[e.key] = false; });

  // Joystick interaction logic
  function getTouchPos(touch) {
    const rect = joystickContainer.getBoundingClientRect();
    return {
      x: touch.clientX - rect.left,
      y: touch.clientY - rect.top
    };
  }

  joystickContainer.addEventListener("touchstart", e => {
    e.preventDefault();
    joystickActive = true;
    moveStick(e.touches[0]);
  }, { passive: false });

  joystickContainer.addEventListener("touchmove", e => {
    e.preventDefault();
    if (joystickActive) {
      moveStick(e.touches[0]);
    }
  }, { passive: false });

  joystickContainer.addEventListener("touchend", e => {
    e.preventDefault();
    joystickActive = false;
    resetStick();
  }, { passive: false });

  function moveStick(touch) {
    const pos = getTouchPos(touch);
    let dx = pos.x - joystickRadius;
    let dy = pos.y - joystickRadius;
    const dist = Math.sqrt(dx*dx + dy*dy);
    const maxDist = joystickRadius - stickRadius;

    if (dist > maxDist) {
      dx = (dx / dist) * maxDist;
      dy = (dy / dist) * maxDist;
    }

    joystickStick.style.left = `${joystickRadius + dx - stickRadius}px`;
    joystickStick.style.top = `${joystickRadius + dy - stickRadius}px`;

    joystickVector.x = dx / maxDist;
    joystickVector.y = dy / maxDist;
  }

  function resetStick() {
    joystickStick.style.left = `${joystickRadius - stickRadius}px`;
    joystickStick.style.top = `${joystickRadius - stickRadius}px`;
    joystickVector.x = 0;
    joystickVector.y = 0;
  }

  // Toggle joystick visibility and controls
  joystickToggle.onclick = () => {
    joystickEnabled = !joystickEnabled;
    saveJoystickSetting();
    updateJoystickVisibility();
  };

  function updateJoystickVisibility() {
    if (joystickEnabled) {
      joystickContainer.style.display = "block";
      touchControls.style.display = "none";
      joystickToggle.textContent = "Disable Joystick";
    } else {
      joystickContainer.style.display = "none";
      touchControls.style.display = "block";
      joystickToggle.textContent = "Enable Joystick";
      joystickVector.x = 0;
      joystickVector.y = 0;
    }
  }

  // Handle window resize
  window.addEventListener("resize", () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
    joystickContainer.style.bottom = "40px";
  });

  // Initialize UI and load data
  window.onload = () => {
    mainMenu.style.display = "block";
    loadGameData();
    coinsDiv.textContent = `Coins: ${orb.coins}`;
    joystickToggle.style.display = "none";
    joystickContainer.style.display = "none";
    touchControls.style.display = "none";
    resetStick();
  };
</script>

</body>
</html>
