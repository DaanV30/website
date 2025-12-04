<!DOCTYPE html>
<html lang="nl">
<head>
<meta charset="UTF-8" />
<title>Geometry Dash Clone</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; }
  body {
    background:#0f172a;
    color:white;
    font-family:sans-serif;
    overflow:hidden;
  }
  canvas {
    display:block;
    background:linear-gradient(#1e293b,#0f172a);
  }

  /* UI */
  #menu {
    position:fixed;
    top:0;left:0;right:0;bottom:0;
    background:rgba(0,0,0,0.65);
    display:flex;
    align-items:center;
    justify-content:center;
    flex-direction:column;
  }
  h1 { font-size:38px; margin-bottom:20px; }
  button {
    padding:10px 22px;
    font-size:18px;
    border:none;
    border-radius:8px;
    margin:8px;
    cursor:pointer;
    font-weight:bold;
    background:#38bdf8;
    color:#0f172a;
    transition:0.15s;
  }
  button:hover { background:#0ea5e9; }
</style>
</head>
<body>

<div id="menu">
  <h1>Geometry Dash Clone</h1>
  <button onclick="startLevel(1)">Level 1</button>
  <button onclick="startLevel(2)">Level 2</button>
  <button onclick="startLevel(3)">Level 3</button>
</div>

<canvas id="game" width="900" height="450"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

let player = {
  x: 80,
  y: 350,
  size: 28,
  dy: 0,
  jump: -13,
  gravity: 0.6,
  grounded: false
};

let obstacles = [];  
let gameSpeed = 6;
let playing = false;
let currentLevel = 1;

function resetPlayer() {
  player.x = 80;
  player.y = 350;
  player.dy = 0;
}

function startLevel(lv) {
  currentLevel = lv;
  obstacles = createLevel(lv);
  resetPlayer();
  playing = true;
  document.getElementById("menu").style.display = "none";
}

// --- LEVEL DESIGN ---
function createLevel(level) {
  let obs = [];

  if (level === 1) {
    obs.push({x:300, y:375, w:30, h:30});
    obs.push({x:500, y:375, w:30, h:30});
    obs.push({x:700, y:375, w:30, h:30});
    obs.push({x:900, y:375, w:40, h:40});
  }

  if (level === 2) {
    obs.push({x:250, y:375, w:35, h:35});
    obs.push({x:450, y:375, w:50, h:50});
    obs.push({x:750, y:375, w:35, h:35});
    obs.push({x:950, y:375, w:40, h:40});
    obs.push({x:1200, y:375, w:50, h:50});
  }

  if (level === 3) {
    obs.push({x:260, y:375, w:35, h:35});
    obs.push({x:380, y:360, w:35, h:50});
    obs.push({x:560, y:375, w:35, h:35});
    obs.push({x:700, y:350, w:35, h:60});
    obs.push({x:950, y:375, w:40, h:40});
    obs.push({x:1200, y:375, w:50, h:50});
    obs.push({x:1500, y:350, w:50, h:60});
  }

  return obs;
}

// --- CONTROLS ---
document.addEventListener("keydown", (e) => {
  if (e.code === "Space") {
    if (player.grounded) {
      player.dy = player.jump;
      player.grounded = false;
    }
  }
});

// --- PHYSICS + COLLISION ---
function update() {
  if (!playing) return;

  player.dy += player.gravity;
  player.y += player.dy;

  if (player.y + player.size > 400) {
    player.y = 400 - player.size;
    player.grounded = true;
  }

  // Move obstacles
  for (let obs of obstacles) {
    obs.x -= gameSpeed;

    // collision
    if (
      player.x < obs.x + obs.w &&
      player.x + player.size > obs.x &&
      player.y < obs.y + obs.h &&
      player.y + player.size > obs.y
    ) {
      die();
    }
  }

  // win condition: last obstacle passed
  if (obstacles.length > 0) {
    let last = obstacles[obstacles.length - 1];
    if (last.x + last.w < 0) win();
  }
}

// --- LOSE ---
function die() {
  playing = false;
  document.getElementById("menu").style.display = "flex";
}

// --- WIN ---
function win() {
  playing = false;
  alert("Level "+currentLevel+" gehaald! 🎉");
  document.getElementById("menu").style.display = "flex";
}

// --- DRAWING ---
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // ground
  ctx.fillStyle = "#334155";
  ctx.fillRect(0, 400, canvas.width, 50);

  // player
  ctx.fillStyle = "#38bdf8";
  ctx.fillRect(player.x, player.y, player.size, player.size);

  // obstacles
  ctx.fillStyle = "#ef4444";
  for (let obs of obstacles) {
    ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
  }
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
