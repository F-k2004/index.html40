
<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🌍 Orbital Simulation</title>
<style>
html,body{
  argin:0;
  overflow:hidden;
  background:#00040a;
  font-family:system-ui,sans-serif;
}
canvas{display:block}
.hud{
  position:absolute;
  right:16px;
  top:16px;
  width:250px;
  padding:14px;
  border-radius:14px;
  background:rgba(255,255,255,0.06);
  backdrop-filter: blur(10px);
  color:#d9f3ff;
  box-shadow:0 0 30px rgba(120,200,255,0.2);
}
.hud h3{margin:0 0 10px;color:#9fdcff;font-size:15px}
.row{display:flex;justify-content:space-between;font-size:13px;margin:6px 0}
.bar{height:6px;background:rgba(255,255,255,0.1);border-radius:4px;overflow:hidden}
.fill{height:100%;background:linear-gradient(90deg,#7fdcff,#ffffff);width:0}
.status{text-align:center;margin-top:8px}
</style>
</head>
<body>

<canvas id="c"></canvas>

<div class="hud">
  <h3>🌍 ORBITAL HUD</h3>
  <div class="row"><span>Altitude</span><span id="alt">0 km</span></div>
  <div class="row"><span>Velocity</span><span id="vel">0 km/s</span></div>
  <div class="bar"><div class="fill" id="velBar"></div></div>
  <div class="row"><span>Thrust</span><span id="thrust">0 mN</span></div>
  <div class="status" id="status">COASTING</div>
</div>

<script>
const canvas = document.getElementById("c");
const ctx = canvas.getContext("2d");
let w,h;
function resize(){w=canvas.width=innerWidth;h=canvas.height=innerHeight}
resize(); addEventListener("resize",resize);

// HUD
const altEl=document.getElementById("alt");
const velEl=document.getElementById("vel");
const velBar=document.getElementById("velBar");
const thrustEl=document.getElementById("thrust");
const statusEl=document.getElementById("status");

// Earth + physics constants (scaled)
const earth={
  x:w/2,
  y:h/2,
  r:60,
  mu:8000 // gravitational parameter (scaled)
};

const ship={
  x:earth.x,
  y:earth.y-180,
  vx:2.3,
  vy:0,
  angle:0
};

let thrusting=false;
addEventListener("mousedown",()=>thrusting=true);
addEventListener("mouseup",()=>thrusting=false);
addEventListener("mousemove",e=>{
  ship.angle=Math.atan2(e.clientY-ship.y,e.clientX-ship.x);
});

function gravity(){
  let dx=earth.x-ship.x;
  let dy=earth.y-ship.y;
  let dist=Math.hypot(dx,dy);
  let force=earth.mu/(dist*dist);
  ship.vx+=force*dx/dist;
  ship.vy+=force*dy/dist;
}

function ionThrust(){
  if(!thrusting) return;
  ship.vx+=Math.cos(ship.angle)*0.03;
  ship.vy+=Math.sin(ship.angle)*0.03;
}

function update(){
  gravity();
  ionThrust();
  ship.x+=ship.vx;
  ship.y+=ship.vy;
}

function drawEarth(){
  ctx.beginPath();
  ctx.arc(earth.x,earth.y,earth.r,0,Math.PI*2);
  ctx.fillStyle="#0b3d91";
  ctx.fill();
  ctx.strokeStyle="#4fc3ff";
  ctx.stroke();
}

function drawShip(){
  ctx.save();
  ctx.translate(ship.x,ship.y);
  ctx.rotate(ship.angle);
  ctx.strokeStyle="#cfefff";
  ctx.lineWidth=2;
  ctx.beginPath();
  ctx.moveTo(12,0);
  ctx.lineTo(-10,-6);
  ctx.lineTo(-6,0);
  ctx.lineTo(-10,6);
  ctx.closePath();
  ctx.stroke();
  ctx.restore();
}

function draw(){
  ctx.fillStyle="rgba(0,4,10,0.3)";
  ctx.fillRect(0,0,w,h);

  update();
  drawEarth();
  drawShip();
  updateHUD();

  requestAnimationFrame(draw);
}

function updateHUD(){
  let dx=ship.x-earth.x;
  let dy=ship.y-earth.y;
  let altitude=Math.hypot(dx,dy)-earth.r;
  let speed=Math.hypot(ship.vx,ship.vy);

  altEl.textContent=(altitude*2).toFixed(0)+" km";
  velEl.textContent=(speed*0.8).toFixed(2)+" km/s";
  velBar.style.width=Math.min(speed/4*100,100)+"%";
  thrustEl.textContent=thrusting?"ON":"OFF";
  statusEl.textContent=thrusting?"ORBIT ADJUST":"STABLE ORBIT";
  statusEl.style.color=thrusting?"#9ff0ff":"#6a8fa3";
}

draw();
</script>
</body>
</html>
