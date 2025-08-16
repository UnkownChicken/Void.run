<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover"/>
<title>Void.run</title>
<style>
  :root {
    --bg: #000;
    --fg: #fff;
    --panel: rgba(255,255,255,0.03);
    --accent: #7ef0ff;
    --font: "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  }
  html,body { height:100%; margin:0; background:var(--bg); color:var(--fg); font-family:var(--font); -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale; }
  #root { position:relative; height:100vh; overflow:hidden; display:flex; align-items:center; justify-content:center; }
  canvas { background: #071018; border-radius:10px; box-shadow: 0 12px 40px rgba(0,0,0,0.6); display:block; }

  /* Main menu */
  #menu {
    position:absolute; inset:0; display:grid; place-items:center; z-index:80;
    background: rgba(0,0,0,1);
  }
  .menu-card { text-align:center; color:var(--fg); }
  .menu-title { font-size:56px; letter-spacing:2px; margin:0 0 24px 0; font-weight:700; }
  .menu-buttons { display:grid; gap:12px; width:260px; margin:0 auto; }
  .menu-button { background:transparent; color:var(--fg); border:2px solid var(--fg); padding:12px 18px; font-size:18px; cursor:pointer; border-radius:8px; font-weight:700; }
  .menu-button:active { transform: translateY(1px); }
  .menu-small { margin-top:12px; font-size:13px; color:rgba(255,255,255,0.6); }

  /* HUD */
  .hud { position:absolute; left:12px; top:12px; z-index:40; color:var(--fg); font-weight:600; font-size:13px; display:flex; gap:12px; flex-wrap:wrap; }
  .panel { background:transparent; border-radius:8px; padding:8px 10px; border:1px solid rgba(255,255,255,0.06); backdrop-filter: blur(3px); }

  /* Controls (top-right) */
  .controls { position:absolute; right:12px; top:12px; z-index:40; display:flex; gap:8px; flex-direction:column; }
  .ctrl-btn { background:transparent; color:var(--fg); border:1px solid rgba(255,255,255,0.08); padding:8px 10px; border-radius:8px; cursor:pointer; font-weight:700; }
  .ctrl-btn:active { transform: translateY(1px); }

  /* Mobile controls */
  #mobileControls { position:absolute; left:50%; transform:translateX(-50%); bottom:18px; display:none; gap:12px; z-index:50; }
  .joy { width:110px; height:110px; border-radius:999px; background:rgba(255,255,255,0.03); display:flex; align-items:center; justify-content:center; touch-action:none; border:1px solid rgba(255,255,255,0.05); }
  .joy-inner { width:34px; height:34px; border-radius:999px; background:rgba(255,255,255,0.08); transform:translate(0,0); }
  .mobile-btn { width:84px; height:56px; background:rgba(255,255,255,0.04); border:1px solid rgba(255,255,255,0.06); border-radius:10px; display:grid; place-items:center; font-weight:800; }

  /* Modals */
  .modal { position: absolute; inset:0; display:none; z-index:90; place-items:center; background: rgba(0,0,0,0.85); padding:16px; }
  .card { background:#071018; border:1px solid rgba(255,255,255,0.06); padding:16px; border-radius:12px; width:min(720px, 92vw); color:var(--fg); }
  .row { display:flex; justify-content:space-between; align-items:center; padding:10px 0; border-bottom:1px dashed rgba(255,255,255,0.05); }
  .row:last-child { border-bottom:none; }
  .grid { display:grid; gap:10px; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); }

  /* Skin tiles */
  .skin { border:1px solid rgba(255,255,255,0.08); padding:10px; border-radius:10px; display:flex; flex-direction:column; gap:8px; align-items:center; text-align:center; }
  .swatch { width:64px; height:64px; border-radius:999px; border:2px solid rgba(255,255,255,0.25); position:relative; overflow:hidden; }
  .skin button { width:100%; border-radius:8px; border:1px solid rgba(255,255,255,0.18); background:transparent; color:var(--fg); padding:6px 8px; cursor:pointer; font-weight:700; }
  .skin .own { opacity:0.8; font-size:12px; }

  /* Toasts */
  #toast { position:absolute; left:50%; transform:translateX(-50%); bottom:22px; background:rgba(0,0,0,0.8); border:1px solid rgba(255,255,255,0.08); padding:10px 14px; border-radius:10px; z-index:95; display:none; font-weight:700; }

  /* small responsive */
  @media (max-width:760px) {
    .menu-title { font-size:40px; }
    #mobileControls { display:flex; }
  }
</style>
<link rel="icon" href="favicon.png">
</head>
<body>
<div id="root">
  <canvas id="game" width="1200" height="700"></canvas>

  <!-- Main Menu -->
  <div id="menu" aria-hidden="false">
    <div class="menu-card">
      <h1 class="menu-title">Void.run</h1>
      <div class="menu-buttons">
        <button class="menu-button" id="btnStart">Start (Survival)</button>
        <button class="menu-button" id="btnEscape">Start Escape Mode</button>
        <button class="menu-button" id="btnHow">How to Play</button>
        <button class="menu-button" id="btnShop">Upgrades</button>
        <button class="menu-button" id="btnSkins">Skin Shop</button>
        <button class="menu-button" id="btnAch">Achievements</button>
      </div>
      <div class="menu-small">Game made by Lemon (Vijay P.)</div>
    </div>
  </div>

  <!-- HUD -->
  <div class="hud" aria-hidden="true">
    <div class="panel">HP: <span id="hp">100</span></div>
    <div class="panel">Score: <span id="score">0</span></div>
    <div class="panel">Coins: <span id="coins">10000</span></div>
    <div class="panel" id="keysHUD" style="display:none">Keys: <span id="keysCount">0</span>/5</div>
    <div class="panel" id="modeHUD">Mode: <span id="modeText">Survival</span></div>
  </div>

  <!-- Controls -->
  <div class="controls" aria-hidden="true">
    <button class="ctrl-btn" id="btnPause">Pause</button>
    <button class="ctrl-btn" id="btnMute">Mute</button>
    <button class="ctrl-btn" id="btnFull">Fullscreen</button>
  </div>

  <!-- Mobile controls -->
  <div id="mobileControls" aria-hidden="true">
    <div class="joy" id="joy"><div class="joy-inner" id="joyInner"></div></div>
    <div class="mobile-btn" id="mobileBoost">BOOST</div>
    <div class="mobile-btn" id="mobileNuke">NUKE</div>
  </div>

  <!-- Upgrades modal (existing shop) -->
  <div id="shopModal" class="modal" role="dialog" aria-modal="true">
    <div class="card">
      <h3 style="margin:0 0 12px 0">Upgrades</h3>
      <div id="shopList"></div>
      <div style="display:flex;justify-content:flex-end;margin-top:12px;gap:8px">
        <button id="closeShop" class="menu-button" style="width:120px">Close</button>
      </div>
    </div>
  </div>

  <!-- Skin Shop modal -->
  <div id="skinModal" class="modal" role="dialog" aria-modal="true">
    <div class="card">
      <h3 style="margin:0 0 12px 0">Skin Shop</h3>
      <div id="skinGrid" class="grid"></div>
      <div style="display:flex;justify-content:space-between;margin-top:12px;align-items:center">
        <div>Coins: <strong id="skinCoins">0</strong></div>
        <button id="closeSkins" class="menu-button" style="width:120px">Close</button>
      </div>
    </div>
  </div>

  <!-- Achievements modal -->
  <div id="achModal" class="modal" role="dialog" aria-modal="true">
    <div class="card">
      <h3 style="margin:0 0 12px 0">Achievements</h3>
      <div id="achList"></div>
      <div style="display:flex;justify-content:flex-end;margin-top:12px;gap:8px">
        <button id="closeAch" class="menu-button" style="width:120px">Close</button>
      </div>
    </div>
  </div>

  <!-- Toast -->
  <div id="toast"></div>
</div>

<script>
(() => {
  // Canvas
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d', { alpha: true });
  let DPR = Math.max(1, Math.min(2, window.devicePixelRatio || 1));
  function resize() {
    const maxW = Math.min(window.innerWidth - 20, 1200);
    const maxH = Math.min(window.innerHeight - 20, 700);
    const ratio = 1200/700;
    let w = maxW, h = Math.round(w / ratio);
    if (h > maxH) { h = maxH; w = Math.round(h * ratio); }
    canvas.style.width = w + 'px';
    canvas.style.height = h + 'px';
    canvas.width = Math.round(w * DPR);
    canvas.height = Math.round(h * DPR);
    ctx.setTransform(DPR,0,0,DPR,0,0);
  }
  window.addEventListener('resize', resize);
  resize();

  // DOM elements
  const menu = document.getElementById('menu');
  const btnStart = document.getElementById('btnStart');
  const btnEscape = document.getElementById('btnEscape');
  const btnHow = document.getElementById('btnHow');
  const btnShop = document.getElementById('btnShop');
  const btnSkins = document.getElementById('btnSkins');
  const btnAch = document.getElementById('btnAch');

  const shopModal = document.getElementById('shopModal');
  const shopList = document.getElementById('shopList');
  const closeShop = document.getElementById('closeShop');

  const skinModal = document.getElementById('skinModal');
  const skinGrid = document.getElementById('skinGrid');
  const closeSkins = document.getElementById('closeSkins');
  const skinCoins = document.getElementById('skinCoins');

  const achModal = document.getElementById('achModal');
  const achList = document.getElementById('achList');
  const closeAch = document.getElementById('closeAch');

  const hpEl = document.getElementById('hp');
  const scoreEl = document.getElementById('score');
  const coinsEl = document.getElementById('coins');
  const btnPause = document.getElementById('btnPause');
  const btnMute = document.getElementById('btnMute');
  const btnFull = document.getElementById('btnFull');

  const keysHUD = document.getElementById('keysHUD');
  const keysCountEl = document.getElementById('keysCount');
  const modeHUD = document.getElementById('modeHUD');
  const modeText = document.getElementById('modeText');

  const mobileControls = document.getElementById('mobileControls');
  const joy = document.getElementById('joy'), joyInner = document.getElementById('joyInner');
  const mobileBoost = document.getElementById('mobileBoost'), mobileNuke = document.getElementById('mobileNuke');

  const toast = document.getElementById('toast');

  const mobile = /Mobi|Android/i.test(navigator.userAgent);
  if (mobile) mobileControls.style.display = 'flex';

  // Game state
  const state = {
    running:false,
    paused:false,
    muted:false,
    lastTS:0,
    score:0,
    currency: Number(localStorage.getItem('void_currency')||0),
    upgrades: JSON.parse(localStorage.getItem('void_upgrades')||'{}'),
    difficulty: 'Normal',
    mode: 'Survival', // 'Survival' | 'Escape'
    lifetimeCoins: Number(localStorage.getItem('void_lifetimeCoins')||0),
    lifetimeTime: Number(localStorage.getItem('void_lifetimeTime')||0),
  };

  function saveState() {
    localStorage.setItem('void_currency', state.currency);
    localStorage.setItem('void_upgrades', JSON.stringify(state.upgrades));
    localStorage.setItem('void_lifetimeCoins', state.lifetimeCoins);
    localStorage.setItem('void_lifetimeTime', state.lifetimeTime);
    localStorage.setItem('void_skins', JSON.stringify(skinsOwned));
    localStorage.setItem('void_skin_selected', selectedSkinId);
    localStorage.setItem('void_ach', JSON.stringify(achStore));
  }

  // Player & entities
  const player = { x:600, y:350, r:12, vx:0, vy:0, speed:240, hp:100, maxHp:100, shield:0, boostUntil:0, alive:true, trail:[], nukes:0 };

  const obstacles = [];
  const orbs = [];
  const particles = [];
  const powerups = [];
  const keysArr = []; // for Escape mode
  let door = null;    // {x,y,r,open}

  // Rendering skin system
  const defaultSkin = { id:'classic', name:'Classic Cyan', price:0, color:'rgba(126,240,255,0.98)', trail:'rgba(126,240,255,0.08)'}; // free
  const skinDefs = [
    defaultSkin,
    { id:'emerald', name:'Emerald', price:120, color:'rgba(120,255,190,0.98)', trail:'rgba(120,255,190,0.09)'},
    { id:'sunset', name:'Sunset', price:150, color:'rgba(255,140,90,0.98)', trail:'rgba(255,140,90,0.09)'},
    { id:'amethyst', name:'Amethyst', price:160, color:'rgba(190,120,255,0.98)', trail:'rgba(190,120,255,0.09)'},
    { id:'void', name:'Deep Void', price:220, color:'rgba(40,180,255,0.98)', trail:'rgba(40,180,255,0.12)'},
    { id:'gold', name:'Auric Core', price:300, color:'rgba(255,215,80,0.98)', trail:'rgba(255,215,80,0.12)'},
  ];
  const skinsOwned = JSON.parse(localStorage.getItem('void_skins')||'{}');
  if (!skinsOwned[defaultSkin.id]) skinsOwned[defaultSkin.id]=true;
  let selectedSkinId = localStorage.getItem('void_skin_selected') || defaultSkin.id;
  function getSkin(){ return skinDefs.find(s=>s.id===selectedSkinId) || defaultSkin; }

  // Achievements (10 hard ones)
  // Keep simple "unlock if condition" checks inside game loop & events
  const achDefs = [
    { id:'score500', name:'Rising Star', desc:'Score 500 in one run', chk: s=> s.runBestScore>=500 },
    { id:'score1500', name:'Void Surfer', desc:'Score 1500 in one run', chk: s=> s.runBestScore>=1500 },
    { id:'survive180', name:'Endurance I', desc:'Survive 180 seconds in a run', chk: s=> s.runTime>=180 },
    { id:'nohithunter', name:'Ghost Frame', desc:'Survive 60s without taking damage', chk: s=> s.noHitTimer>=60 },
    { id:'nukemaster', name:'Nuclear Option', desc:'Destroy 25 enemies with NUKE (lifetime)', chk: s=> s.lifetimeNukeKills>=25 },
    { id:'rich1000', name:'Tycoon', desc:'Accumulate 1000 lifetime coins', chk: s=> state.lifetimeCoins>=1000 },
    { id:'boost50', name:'Flash Step', desc:'Use BOOST 50 times (lifetime)', chk: s=> s.lifetimeBoosts>=50 },
    { id:'shopper', name:'First Purchase', desc:'Buy any upgrade or skin', chk: s=> s.anyPurchase },
    { id:'escapist', name:'Break Out', desc:'Win Escape Mode once', chk: s=> s.escapeWins>=1 },
    { id:'speedrunner', name:'Key Rush', desc:'Collect all 5 keys in under 90s', chk: s=> s.lastEscapeTime>0 && s.lastEscapeTime<=90 },
  ];
  const achStore = JSON.parse(localStorage.getItem('void_ach')||'{}'); // {id:true}
  const achRuntime = {
    runTime:0, runBestScore:0, noHitTimer:0, lastHit:0,
    lifetimeNukeKills: Number(localStorage.getItem('void_lifeNukeKills')||0),
    lifetimeBoosts: Number(localStorage.getItem('void_lifeBoosts')||0),
    anyPurchase: !!localStorage.getItem('void_anyPurchase'),
    escapeWins: Number(localStorage.getItem('void_escapeWins')||0),
    lastEscapeTime: Number(localStorage.getItem('void_lastEscapeTime')||0),
  };
  function persistAchRuntime(){
    localStorage.setItem('void_lifeNukeKills', achRuntime.lifetimeNukeKills);
    localStorage.setItem('void_lifeBoosts', achRuntime.lifetimeBoosts);
    localStorage.setItem('void_anyPurchase', achRuntime.anyPurchase ? '1':'');
    localStorage.setItem('void_escapeWins', achRuntime.escapeWins);
    localStorage.setItem('void_lastEscapeTime', achRuntime.lastEscapeTime);
  }
  function tryUnlockAchievements(){
    for (const a of achDefs){
      if (!achStore[a.id] && a.chk(achRuntime)){ achStore[a.id]=true; toastMsg('Achievement Unlocked: ' + a.name); }
    }
    saveState();
  }

  // UI helpers
  function updateHUD() {
    hpEl.textContent = Math.max(0, Math.round(player.hp));
    scoreEl.textContent = Math.floor(state.score);
    coinsEl.textContent = state.currency;
    modeText.textContent = state.mode;
    keysHUD.style.display = state.mode==='Escape' ? 'inline-flex' : 'none';
  }
  function toastMsg(txt){
    toast.textContent = txt;
    toast.style.display = 'block';
    clearTimeout(window._toastT);
    window._toastT = setTimeout(()=> toast.style.display='none', 1800);
  }

  // Input
  const keys = {};
  window.addEventListener('keydown', e => {
    const k = e.key.toLowerCase();
    keys[k] = true;
    if (e.key === ' ') e.preventDefault();
    if (k === 'shift') playerBoost();
    if (k === 'n') activateNuke();
  });
  window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

  // Mobile joystick
  let joyActive=false, joyRect=null;
  let joyState={x:0,y:0};
  function joyStart(e){
    joyRect = joy.getBoundingClientRect();
    joyActive = true;
    processJoy(e.touches ? e.touches[0] : e);
  }
  function joyMove(e){
    if (!joyActive) return;
    processJoy(e.touches ? e.touches[0] : e);
  }
  function joyEnd(){ joyActive=false; joyInner.style.transform='translate(0,0)'; joyState={x:0,y:0}; }
  function processJoy(p){
    const cx = joyRect.left + joyRect.width/2, cy = joyRect.top + joyRect.height/2;
    const dx = p.clientX - cx, dy = p.clientY - cy;
    const max = joyRect.width/2 - 12;
    const nx = Math.max(-1, Math.min(1, dx/max)), ny = Math.max(-1, Math.min(1, dy/max));
    joyState.x = nx; joyState.y = ny;
    joyInner.style.transform = `translate(${nx*28}px, ${ny*28}px)`;
  }
  joy.addEventListener('pointerdown', joyStart);
  window.addEventListener('pointermove', joyMove);
  window.addEventListener('pointerup', joyEnd);
  joy.addEventListener('touchstart', joyStart, {passive:false});
  joy.addEventListener('touchmove', joyMove, {passive:false});
  joy.addEventListener('touchend', joyEnd);
  mobileBoost.addEventListener('click', ()=> playerBoost());
  mobileNuke.addEventListener('click', ()=> activateNuke());

  // Helpers
  const rand = (a,b) => Math.random()*(b-a)+a;
  const clamp = (v,a,b) => Math.max(a, Math.min(b,v));
  const circleColl = (a,b) => { const dx=a.x-b.x, dy=a.y-b.y, r=(a.r||0)+(b.r||0); return dx*dx+dy*dy <= r*r; };

  // Spawning
  let elapsed = 0, spawnTimer = 0;
  function spawnObstacle(type) {
    const w = canvas.width/DPR, h = canvas.height/DPR;
    const edge = Math.floor(rand(0,4));
    let x,y,vx,vy;
    if (edge===0){ x=-40; y=rand(0,h); vx=rand(100,200); vy=rand(-60,60); }
    if (edge===1){ x=w+40; y=rand(0,h); vx=rand(-200,-100); vy=rand(-60,60); }
    if (edge===2){ x=rand(0,w); y=-40; vx=rand(-60,60); vy=rand(100,200); }
    if (edge===3){ x=rand(0,w); y=h+40; vx=rand(-60,60); vy=rand(-200,-100); }
    const obj = { type, x, y, vx, vy, r: 18, hp:1, angle:rand(0,Math.PI*2), spin: rand(-2,2) };
    if (type==='homing'){ obj.r=16; obj.speed=140; }
    if (type==='big'){ obj.r=36; obj.hp=3; obj.vx *= 0.6; obj.vy *= 0.6; }
    if (type==='spinner'){ obj.r=20; obj.spin = rand(-6,6); }
    if (type==='splitter'){ obj.r=24; obj.hp=2; }
    obstacles.push(obj);
  }
  function spawnOrb(){ const w = canvas.width/DPR, h = canvas.height/DPR; orbs.push({ x:rand(60,w-60), y:rand(60,h-60), r:8, value:1, born:performance.now() }); }
  function spawnPowerup(){ const types=['shield','boost','slow','nuke']; powerups.push({ x:rand(60,canvas.width/DPR-60), y:rand(60,canvas.height/DPR-60), type:types[Math.floor(rand(0,types.length))], r:12, born:performance.now() }); }

  function spawnKey(){
    const w = canvas.width/DPR, h = canvas.height/DPR;
    keysArr.push({ x:rand(60,w-60), y:rand(60,h-60), r:10, born:performance.now() });
    updateKeyHUD();
  }
  function spawnDoor(){
    const w = canvas.width/DPR, h = canvas.height/DPR;
    const edge = Math.floor(rand(0,4));
    let x = (edge%2===0) ? rand(40, w-40) : (edge===1 ? w-40 : 40);
    let y = (edge%2===1) ? rand(40, h-40) : (edge===2 ? h-40 : 40);
    door = { x, y, r:16, open:true };
  }
  function updateKeyHUD(){ keysCountEl.textContent = (5 - keysArr.length < 5) ? (5 - Math.max(0, 5 - keysCollected)) : keysCollected; } // replaced later

  // Player actions
  function playerBoost(){
    if (!player.alive) return;
    const now = performance.now();
    player.boostUntil = now + (state.upgrades['boost'] ? 900 : 450);
    achRuntime.lifetimeBoosts++; persistAchRuntime();
    if (!state.muted) playSfx('boost');
    tryUnlockAchievements();
  }
  function activateNuke(){
    let count = 0;
    for (let i=obstacles.length-1;i>=0;i--){
      if (Math.hypot(obstacles[i].x-player.x, obstacles[i].y-player.y) < 220){
        state.score += 15;
        obstacles.splice(i,1); count++;
      }
    }
    if (count){ if (!state.muted) playSfx('nuke'); achRuntime.lifetimeNukeKills += count; persistAchRuntime(); tryUnlockAchievements(); }
  }

  // Simple sfx (synth)
  function playSfx(n){
    if (state.muted) return;
    try {
      const ctxAudio = (window._audioCtx ||= new (window.AudioContext || window.webkitAudioContext)());
      const t = ctxAudio.currentTime;
      const o = ctxAudio.createOscillator();
      const g = ctxAudio.createGain();
      if (n==='boost'){ o.type='sawtooth'; o.frequency.setValueAtTime(420,t); g.gain.setValueAtTime(0.0025,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); o.frequency.exponentialRampToValueAtTime(120, t+0.18); g.gain.exponentialRampToValueAtTime(0.0001, t+0.18); o.stop(t+0.18); }
      if (n==='hit'){ o.type='square'; o.frequency.setValueAtTime(120,t); g.gain.setValueAtTime(0.02,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); g.gain.exponentialRampToValueAtTime(0.0001, t+0.14); o.stop(t+0.14); }
      if (n==='nuke'){ o.type='sine'; o.frequency.setValueAtTime(180,t); g.gain.setValueAtTime(0.03,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); g.gain.exponentialRampToValueAtTime(0.0001, t+0.6); o.stop(t+0.6); }
      if (n==='orb'){ o.type='triangle'; o.frequency.setValueAtTime(660,t); g.gain.setValueAtTime(0.008,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); g.gain.exponentialRampToValueAtTime(0.0001, t+0.09); o.stop(t+0.12); }
      if (n==='key'){ o.type='triangle'; o.frequency.setValueAtTime(520,t); g.gain.setValueAtTime(0.01,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); o.frequency.exponentialRampToValueAtTime(880, t+0.12); g.gain.exponentialRampToValueAtTime(0.0001, t+0.15); o.stop(t+0.18); }
      if (n==='win'){ o.type='sawtooth'; o.frequency.setValueAtTime(340,t); g.gain.setValueAtTime(0.02,t); o.connect(g); g.connect(ctxAudio.destination); o.start(t); o.frequency.exponentialRampToValueAtTime(720, t+0.6); g.gain.exponentialRampToValueAtTime(0.0001, t+0.7); o.stop(t+0.7); }
    } catch(e){}
  }

  // Main update & render
  let keysCollected = 0;
  function startGame(mode='Survival') {
    state.mode = mode;
    keysCollected = 0; keysArr.length = 0; door = null;
    state.running = true; state.paused = false; elapsed = 0; spawnTimer = 0;
    obstacles.length = 0; orbs.length = 0; particles.length = 0; powerups.length = 0;
    const extraHp = (state.upgrades['extraHp']||0) * 20;
    player.maxHp = 100 + extraHp;
    player.x = canvas.width/2/DPR; player.y = canvas.height/2/DPR; player.vx=0; player.vy=0; player.hp = player.maxHp; player.shield= (state.upgrades['startShield']||0) * 50; player.alive=true; player.trail=[];
    player.nukes = (state.upgrades['nukeItem']||0);
    modeText.textContent = state.mode;
    keysHUD.style.display = (state.mode==='Escape') ? 'inline-flex' : 'none';
    keysCountEl.textContent = 0;
    achRuntime.runTime = 0; achRuntime.noHitTimer = 0; achRuntime.runBestScore = 0;
    menu.style.display = 'none';
    state.score = 0;
    state.lastTS = performance.now();
    if (state.mode==='Escape'){ spawnKey(); } // start with a key on map
    requestAnimationFrame(loop);
    updateHUD();
  }

  function endGame(win=false) {
    state.running = false;
    player.alive = false;
    menu.style.display = 'grid';
    if (win){ toastMsg('You escaped! +50 coins'); state.currency += 50; state.lifetimeCoins += 50; achRuntime.escapeWins++; persistAchRuntime(); tryUnlockAchievements(); }
    saveState();
    updateHUD();
  }

  function loop(){
    if (!state.running) return;
    const now = performance.now();
    const dt = Math.min(40, now - state.lastTS)/1000;
    state.lastTS = now;
    if (!state.paused){ update(dt); }
    render();
    requestAnimationFrame(loop);
  }

  function update(dt){
    elapsed += dt;
    achRuntime.runTime += dt;
    state.lifetimeTime += dt;

    spawnTimer += dt;

    // spawn logic with two new enemy types sprinkled in
    const desiredInterval = clamp(1 - Math.min(0.7, elapsed*0.005), 0.25, 1.2);
    if (spawnTimer > desiredInterval){
      spawnTimer = 0;
      const r = Math.random();
      if (elapsed > 25 && r < 0.10) spawnObstacle('big');
      else if (elapsed > 18 && r < 0.20) spawnObstacle('spinner');
      else if (elapsed > 12 && r < 0.30) spawnObstacle('homing');
      else if (elapsed > 35 && r < 0.36) spawnObstacle('splitter');
      else spawnObstacle('basic');
    }
    if (Math.random() < 0.012 * dt * 60) spawnOrb();
    if (Math.random() < 0.002 * dt * 60) spawnPowerup();

    // Escape mode pacing
    if (state.mode==='Escape'){
      // occasionally ensure a key exists until 5 collected
      if (keysCollected < 5 && keysArr.length < 1 && Math.random() < 0.002 * dt * 60) spawnKey();
    }

    // handle input
    let mvx=0,mvy=0;
    if (mobile && (Math.abs(joyState.x) > 0.02 || Math.abs(joyState.y) > 0.02)) { mvx += joyState.x; mvy += joyState.y; }
    if (keys['arrowup']||keys['w']) mvy -= 1;
    if (keys['arrowdown']||keys['s']) mvy += 1;
    if (keys['arrowleft']||keys['a']) mvx -= 1;
    if (keys['arrowright']||keys['d']) mvx += 1;
    const len = Math.hypot(mvx,mvy);
    if (len>0){ mvx/=len; mvy/=len; }

    const boosting = performance.now() < player.boostUntil;
    const speed = player.speed * (boosting ? 2.4 : 1);
    player.vx += (mvx*speed - player.vx) * clamp(12*dt,0,1);
    player.vy += (mvy*speed - player.vy) * clamp(12*dt,0,1);
    player.x += player.vx * dt;
    player.y += player.vy * dt;

    const w = canvas.width/DPR, h = canvas.height/DPR;
    player.x = clamp(player.x, 10, w-10);
    player.y = clamp(player.y, 10, h-10);

    // trail
    player.trail.unshift({x:player.x,y:player.y,age:0});
    if (player.trail.length > 40) player.trail.pop();
    player.trail.forEach(t => t.age += dt*1000);

    // move obstacles
    for (let i=obstacles.length-1;i>=0;i--){
      const o = obstacles[i];
      if (o.type==='homing'){
        const ang = Math.atan2(player.y - o.y, player.x - o.x);
        o.vx += Math.cos(ang) * 20 * dt;
        o.vy += Math.sin(ang) * 20 * dt;
      }
      if (o.type==='spinner'){
        o.angle += o.spin * dt;
        o.vx += Math.cos(o.angle) * 6 * dt;
        o.vy += Math.sin(o.angle) * 6 * dt;
      }
      o.x += o.vx * dt; o.y += o.vy * dt;
      if (o.x < -220 || o.x > w+220 || o.y < -220 || o.y > h+220) obstacles.splice(i,1);
    }

    // orbs pickup
    for (let i=orbs.length-1;i>=0;i--){
      const o = orbs[i];
      if (performance.now() - o.born > 30000) { orbs.splice(i,1); continue; }
      if (circleColl(o, player)){
        state.score += 5;
        state.currency += 1;
        state.lifetimeCoins += 1;
        orbs.splice(i,1);
        if (!state.muted) playSfx('orb');
        saveState();
      }
    }

    // powerup pickup
    for (let i=powerups.length-1;i>=0;i--){
      const p = powerups[i];
      if (circleColl(p, player)){
        if (p.type==='shield') player.shield = Math.min(100, player.shield + 40);
        if (p.type==='boost') player.boostUntil = performance.now() + 1400;
        if (p.type==='slow') obstacles.forEach(o => { o.vx *= 0.35; o.vy *= 0.35; });
        if (p.type==='nuke') activateNuke();
        powerups.splice(i,1);
      }
    }

    // keys pickup & door
    if (state.mode==='Escape'){
      for (let i=keysArr.length-1;i>=0;i--){
        const k = keysArr[i];
        if (circleColl(k, player)){
          keysArr.splice(i,1);
          keysCollected++;
          keysCountEl.textContent = keysCollected;
          if (!state.muted) playSfx('key');
          if (keysCollected >= 5 && !door){ spawnDoor(); }
        }
      }
      if (door && circleColl(door, player)){
        achRuntime.lastEscapeTime = achRuntime.runTime;
        persistAchRuntime(); tryUnlockAchievements();
        if (!state.muted) playSfx('win');
        endGame(true);
        return;
      }
    }

    // collisions
    let tookDamageThisFrame = false;
    for (let i=obstacles.length-1;i>=0;i--){
      const o = obstacles[i];
      if (circleColl(o, player)){
        if (player.shield > 0){
          player.shield -= 28;
          if (!state.muted) playSfx('hit');
          if (o.hp > 1){ o.hp--; } else { obstacles.splice(i,1); }
        } else {
          player.hp -= o.r * 0.9 * (o.type==='big' ? 1.5 : 1);
          if (!state.muted) playSfx('hit');
          tookDamageThisFrame = true;
          if (o.hp > 1){ 
            o.hp--; 
            if (o.type==='splitter' && o.hp===1){
              // split into two smaller basics
              for (let s=0;s<2;s++){
                obstacles.push({ type:'basic', x:o.x, y:o.y, vx:rand(-160,160), vy:rand(-160,160), r:14, hp:1, angle:0 });
              }
            }
          } else { obstacles.splice(i,1); }
          if (player.hp <= 0){ endGame(false); return; }
        }
      }
    }
    if (tookDamageThisFrame){ achRuntime.noHitTimer = 0; } else { achRuntime.noHitTimer += dt; }

    // particles update
    for (let i=particles.length-1;i>=0;i--){
      particles[i].age += dt*1000;
      particles[i].x += (particles[i].dx || 0) * dt;
      particles[i].y += (particles[i].dy || 0) * dt;
      if (particles[i].age > (particles[i].life || 600)) particles.splice(i,1);
    }

    // scoring & achievements
    state.score += dt * (1 + (performance.now() < player.boostUntil ? 6 : 0));
    achRuntime.runBestScore = Math.max(achRuntime.runBestScore, Math.floor(state.score));
    tryUnlockAchievements();

    updateHUD();
    saveState();
  }

  function render(){
    const w = canvas.width/DPR, h = canvas.height/DPR;
    ctx.clearRect(0,0, w, h );
    // background grid glow
    ctx.fillStyle = '#071018';
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 0.12;
    ctx.strokeStyle = 'rgba(126,240,255,0.25)';
    ctx.lineWidth = 1;
    for (let gx=0; gx<w; gx+=60){ ctx.beginPath(); ctx.moveTo(gx,0); ctx.lineTo(gx,h); ctx.stroke(); }
    for (let gy=0; gy<h; gy+=60){ ctx.beginPath(); ctx.moveTo(0,gy); ctx.lineTo(w,gy); ctx.stroke(); }
    ctx.globalAlpha = 1;

    // orbs
    for (const o of orbs){
      ctx.beginPath();
      ctx.fillStyle = 'rgba(126,240,255,0.9)';
      ctx.arc(o.x,o.y,o.r,0,Math.PI*2); ctx.fill();
    }

    // powerups
    for (const p of powerups){
      ctx.beginPath();
      ctx.fillStyle = p.type==='shield' ? 'rgba(124,255,200,0.9)' : p.type==='boost' ? 'rgba(126,240,255,0.9)' : p.type==='slow' ? 'rgba(255,220,120,0.9)' : 'rgba(255,120,140,0.9)';
      ctx.arc(p.x,p.y,p.r,0,Math.PI*2); ctx.fill();
    }

    // keys (Escape mode)
    if (state.mode==='Escape'){
      for (const k of keysArr){
        ctx.beginPath();
        ctx.fillStyle = 'rgba(255,230,120,0.95)';
        ctx.arc(k.x,k.y,k.r,0,Math.PI*2); ctx.fill();
        // key notch
        ctx.fillRect(k.x-2,k.y-12,4,6);
      }
      if (door){
        ctx.beginPath();
        ctx.strokeStyle = 'rgba(255,255,255,0.85)';
        ctx.lineWidth = 4;
        ctx.arc(door.x,door.y,door.r+8,0,Math.PI*2); ctx.stroke();
        ctx.beginPath();
        ctx.fillStyle = 'rgba(255,255,255,0.9)';
        ctx.arc(door.x,door.y,door.r,0,Math.PI*2); ctx.fill();
      }
    }

    // obstacles
    for (const o of obstacles){
      ctx.beginPath();
      ctx.fillStyle = o.type==='homing' ? 'rgba(255,95,140,0.95)' : o.type==='big' ? 'rgba(255,110,120,0.9)' : o.type==='spinner' ? 'rgba(255,170,120,0.95)' : o.type==='splitter' ? 'rgba(255,120,200,0.95)' : 'rgba(255,120,130,0.95)';
      ctx.arc(o.x,o.y,o.r,0,Math.PI*2); ctx.fill();
    }

    // player trail (skin-colored)
    const skin = getSkin();
    for (let i=player.trail.length-1;i>=0;i--){
      const t = player.trail[i];
      const a = 1 - (i / player.trail.length);
      ctx.beginPath();
      ctx.fillStyle = skin.trail.replace(/0\.\d+\)/, (m)=> m) || 'rgba(126,240,255,0.08)';
      ctx.globalAlpha = 0.06 * a + 0.02;
      ctx.arc(t.x,t.y, 8 * (a*0.8), 0, Math.PI*2);
      ctx.fill();
      ctx.globalAlpha = 1;
    }

    // player
    ctx.beginPath();
    ctx.fillStyle = skin.color;
    ctx.arc(player.x, player.y, player.r, 0, Math.PI*2); ctx.fill();

    // shield
    if (player.shield > 0){
      ctx.beginPath();
      ctx.strokeStyle = `rgba(124,255,200,${0.08 + (player.shield/100)*0.4})`;
      ctx.lineWidth = 6; ctx.arc(player.x, player.y, player.r+8, 0, Math.PI*2); ctx.stroke();
    }
  }

  // Buttons behaviour
  btnStart.addEventListener('click', ()=> { startGame('Survival'); });
  btnEscape.addEventListener('click', ()=> { startGame('Escape'); });

  btnHow.addEventListener('click', ()=> {
    alert(
`How to play:
• Move with WASD or arrow keys (or use joystick on mobile).
• Hold/press SHIFT for BOOST. Press N for NUKE near enemies.
• Collect orbs for score & coins. Buy upgrades and skins in the shops.
• Avoid obstacles — shield & powerups help.

Escape Mode:
• Collect 5 golden keys scattered around the arena.
• After 5 keys, a DOOR appears on an edge — reach it to escape and win!
• Winning grants bonus coins. Keys are timed—go fast for the "Key Rush" achievement.`);
  });

  btnShop.addEventListener('click', ()=> { populateShop(); shopModal.style.display = 'grid'; });
  closeShop.addEventListener('click', ()=> { shopModal.style.display = 'none'; });

  btnSkins.addEventListener('click', ()=> { populateSkins(); skinModal.style.display = 'grid'; });
  closeSkins.addEventListener('click', ()=> { skinModal.style.display = 'none'; });

  btnAch.addEventListener('click', ()=> { populateAchievements(); achModal.style.display = 'grid'; });
  closeAch.addEventListener('click', ()=> { achModal.style.display = 'none'; });

  btnPause.addEventListener('click', ()=> { state.paused = !state.paused; btnPause.textContent = state.paused ? 'Resume' : 'Pause'; });
  btnMute.addEventListener('click', ()=> { state.muted = !state.muted; btnMute.textContent = state.muted ? 'Unmute' : 'Mute'; });
  btnFull.addEventListener('click', ()=> {
    if (!document.fullscreenElement) document.documentElement.requestFullscreen().catch(()=>{});
    else document.exitFullscreen();
  });

  // Upgrades shop (unchanged logic, refreshed layout)
  const shopItems = [
    { id:'shield', label:'Start Shield +50', cost:40, apply: () => { state.upgrades['startShield'] = (state.upgrades['startShield']||0) + 1; } },
    { id:'boost', label:'Better Boost (longer)', cost:45, apply: () => { state.upgrades['boost'] = (state.upgrades['boost']||0) + 1; } },
    { id:'extraHp', label:'Max HP +20', cost:70, apply: () => { state.upgrades['extraHp'] = (state.upgrades['extraHp']||0) + 1; } },
    { id:'nukeItem', label:'Start with 1 Nuke', cost:110, apply: () => { state.upgrades['nukeItem'] = (state.upgrades['nukeItem']||0) + 1; } }
  ];

  function populateShop(){
    shopList.innerHTML = '';
    shopItems.forEach(it => {
      const row = document.createElement('div');
      row.className = 'row';
      row.innerHTML = `<div>${it.label}</div><div><strong>${it.cost}</strong> <button data-id="${it.id}">Buy</button></div>`;
      shopList.appendChild(row);
      row.querySelector('button').addEventListener('click', ()=> {
        if (state.currency >= it.cost) {
          state.currency -= it.cost;
          state.lifetimeCoins -= it.cost < 0 ? 0 : 0; // lifetime is earn-only
          it.apply();
          achRuntime.anyPurchase = true; persistAchRuntime();
          saveState();
          toastMsg('Purchased: ' + it.label);
          shopModal.style.display = 'none';
          updateHUD();
          tryUnlockAchievements();
        } else {
          alert('Not enough coins');
        }
      });
    });
    const cur = document.createElement('div'); cur.style.marginTop='10px'; cur.textContent = 'Coins: ' + state.currency; shopList.appendChild(cur);
  }

  // Skin Shop
  function populateSkins(){
    skinCoins.textContent = state.currency;
    skinGrid.innerHTML = '';
    skinDefs.forEach(s=>{
      const el = document.createElement('div');
      el.className = 'skin';
      el.innerHTML = `
        <div class="swatch" style="background:${s.color}; box-shadow: inset 0 0 18px rgba(0,0,0,0.45)"></div>
        <div style="font-weight:800">${s.name}</div>
        <div class="own">${skinsOwned[s.id] ? 'Owned' : s.price + ' coins'}</div>
        <button>${selectedSkinId===s.id ? 'Selected' : (skinsOwned[s.id] ? 'Equip' : 'Buy')}</button>
      `;
      const btn = el.querySelector('button');
      btn.addEventListener('click', ()=>{
        if (skinsOwned[s.id]) {
          selectedSkinId = s.id;
          saveState();
          populateSkins();
          toastMsg('Equipped: ' + s.name);
        } else {
          if (state.currency >= s.price){
            state.currency -= s.price;
            skinsOwned[s.id] = true;
            achRuntime.anyPurchase = true; persistAchRuntime();
            selectedSkinId = s.id;
            saveState();
            populateSkins();
            updateHUD();
            toastMsg('Purchased: ' + s.name);
            tryUnlockAchievements();
          } else {
            alert('Not enough coins');
          }
        }
      });
      skinGrid.appendChild(el);
    });
  }

  // Achievements modal
  function populateAchievements(){
    achList.innerHTML = '';
    const total = achDefs.length;
    const unlocked = achDefs.filter(a=>achStore[a.id]).length;
    const header = document.createElement('div');
    header.style.marginBottom='8px';
    header.textContent = `Unlocked ${unlocked}/${total}`;
    achList.appendChild(header);

    achDefs.forEach(a=>{
      const row = document.createElement('div');
      row.className='row';
      row.innerHTML = `
        <div>
          <div style="font-weight:800">${a.name}</div>
          <div style="opacity:0.75;font-size:12px">${a.desc}</div>
        </div>
        <div style="font-weight:800; ${achStore[a.id]?'color:#7ef0ff':''}">${achStore[a.id] ? 'Unlocked' : 'Locked'}</div>
      `;
      achList.appendChild(row);
    });
  }

  // pre-warm audio on first interaction
  window.addEventListener('pointerdown', ()=> { if (!window._audioWarm) { try{ (window._audioCtx ||= new (window.AudioContext || window.webkitAudioContext)()).resume(); }catch(e){} window._audioWarm=true; } }, {once:true});

  // Expose for debugging
  window.VoidRun = { startGame, endGame: () => { endGame(); }, state };

  // initial HUD
  updateHUD();
})();
</script>
</body>
</html>
