#<!DOCTYPE html><html lang="ar">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no" />
<title>Cat Fiappy</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    font-family: Arial;
    background: #87ceeb;
    touch-action: manipulation;
  }
  canvas { display: block; }#menu { position: fixed; inset: 0; background-image: url('background.png'); background-size: cover; background-position: center; display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 10; }

#menuContent { text-align: center; animation: float 2.5s ease-in-out infinite; }

h1 { font-size: 46px; color: white; text-shadow: 2px 2px 6px black; margin-bottom: 20px; }

#difficulty { color: white; font-size: 18px; margin-bottom: 20px; }

button { font-size: 22px; padding: 14px 40px; border: none; border-radius: 12px; cursor: pointer; }

#credit { position: absolute; bottom: 15px; color: white; font-size: 16px; text-decoration: none; text-shadow: 1px 1px 4px black; }

@keyframes float { 0% { transform: translateY(0); } 50% { transform: translateY(-10px); } 100% { transform: translateY(0); } } </style>

</head>
<body><div id="menu">
  <div id="menuContent">
    <h1>🐱 Cat Fiappy</h1>
    <div id="difficulty">
      <label><input type="radio" name="diff" value="easy" checked> سهل</label><br>
      <label><input type="radio" name="diff" value="medium"> متوسط</label><br>
      <label><input type="radio" name="diff" value="hard"> صعب</label>
    </div>
    <button id="startBtn">ابدأ اللعبة</button>
  </div>
  <a id="credit" href="https://www.instagram.com/_jw16" target="_blank">karam almahayni</a>
</div><canvas id="game"></canvas>

<script>
// عناصر
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const menu = document.getElementById('menu');
const startBtn = document.getElementById('startBtn');

// حجم الشاشة
function resize() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resize);
resize();

// متغيرات اللعبة
let gravity = 0.6;
let jump = -10;
let speed = 2;
let time = 0;
let running = false;
let pipeGap = 260;
let selectedDifficulty = 'easy';

const cat = {
  x: canvas.width * 0.25,
  y: canvas.height / 2,
  r: 18,
  vy: 0
};

let pipes = [];
const pipeWidth = 60;
const pipeDistance = 320;

function resetGame() {
  pipes = [];
  cat.y = canvas.height / 2;
  cat.vy = 0;
  speed = 2;
  time = 0;
}

// زر البداية (مضمون)
startBtn.onclick = () => {
  const diff = document.querySelector('input[name="diff"]:checked').value;

  if (diff === 'easy') pipeGap = 300;
  if (diff === 'medium') pipeGap = 260;
  if (diff === 'hard') pipeGap = 220;

  selectedDifficulty = diff;
  resetGame();
  running = true;
  menu.style.display = 'none';
};

// القفز
function jumpCat() {
  if (running) cat.vy = jump;
}
canvas.addEventListener('click', jumpCat);
canvas.addEventListener('touchstart', e => {
  e.preventDefault();
  jumpCat();
});

function addPipe() {
  const minTop = 80;
  const maxTop = canvas.height - pipeGap - 120;
  const top = Math.random() * (maxTop - minTop) + minTop;
  pipes.push({ x: canvas.width + pipeWidth, top });
}

function update(dt) {
  if (!running) return;

  time += dt / 1000;
  speed = 2 + time * 0.15;

  cat.vy += gravity;
  cat.y += cat.vy;

  if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - pipeDistance) {
    addPipe();
  }

  pipes.forEach(p => p.x -= speed);
  pipes = pipes.filter(p => p.x + pipeWidth > 0);

  for (let p of pipes) {
    if (
      cat.x + cat.r > p.x &&
      cat.x - cat.r < p.x + pipeWidth &&
      (cat.y - cat.r < p.top || cat.y + cat.r > p.top + pipeGap)
    ) {
      running = false;
      menu.style.display = 'flex';
    }
  }

  if (cat.y < 0 || cat.y > canvas.height) {
    running = false;
    menu.style.display = 'flex';
  }
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // القطة
  ctx.fillStyle = '#ffcc99';
  ctx.beginPath();
  ctx.arc(cat.x, cat.y, cat.r, 0, Math.PI * 2);
  ctx.fill();

  // العواميد
  ctx.fillStyle = '#2e8b57';
  pipes.forEach(p => {
    ctx.fillRect(p.x, 0, pipeWidth, p.top);
    ctx.fillRect(p.x, p.top + pipeGap, pipeWidth, canvas.height);
  });

  // الوقت + الصعوبة
  ctx.fillStyle = '#000';
  ctx.font = '20px Arial';
  const mins = Math.floor(time / 60);
  const secs = Math.floor(time % 60);
  ctx.fillText(`الوقت: ${mins}:${secs.toString().padStart(2,'0')}`, 10, 30);

  const diffText = selectedDifficulty === 'easy' ? 'سهل' : selectedDifficulty === 'medium' ? 'متوسط' : 'صعب';
  ctx.fillText(`الصعوبة: ${diffText}`, 10, 55);
}

let last = performance.now();
function loop(now) {
  const dt = now - last;
  last = now;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script></body>
</html>
