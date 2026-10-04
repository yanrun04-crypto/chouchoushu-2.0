<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<title>孤岛臭臭鼠快跑</title>

<style>
*{
  box-sizing:border-box;
  -webkit-tap-highlight-color:transparent;
}
html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#07151b;
  font-family:-apple-system,BlinkMacSystemFont,"PingFang SC","Microsoft YaHei",sans-serif;
}
canvas{
  display:block;
  width:100vw;
  height:100vh;
  touch-action:none;
}
#hud{
  position:fixed;
  top:calc(12px + env(safe-area-inset-top));
  left:14px;
  right:14px;
  display:flex;
  justify-content:space-between;
  align-items:flex-start;
  color:white;
  z-index:10;
  pointer-events:none;
}
.box{
  padding:8px 12px;
  border-radius:14px;
  background:rgba(0,0,0,.28);
  backdrop-filter:blur(8px);
  border:1px solid rgba(255,255,255,.15);
  text-shadow:0 2px 4px #000;
}
.big{
  font-size:20px;
  font-weight:800;
}
.small{
  font-size:12px;
  opacity:.75;
}
#intro{
  position:fixed;
  left:50%;
  top:34%;
  transform:translate(-50%,-50%);
  color:white;
  font-size:25px;
  font-weight:800;
  text-shadow:0 3px 10px #000;
  opacity:0;
  z-index:20;
  pointer-events:none;
  white-space:nowrap;
}
#start,#over{
  position:fixed;
  inset:0;
  z-index:30;
  display:flex;
  align-items:center;
  justify-content:center;
  text-align:center;
  color:white;
  background:linear-gradient(
    rgba(0,15,20,.18),
    rgba(0,15,20,.65)
  );
}
.card{
  width:min(88vw,420px);
  padding:28px 22px;
  border-radius:28px;
  background:rgba(8,24,30,.72);
  backdrop-filter:blur(14px);
  border:1px solid rgba(255,255,255,.16);
  box-shadow:0 20px 70px rgba(0,0,0,.4);
}
.title{
  font-size:34px;
  font-weight:900;
  letter-spacing:2px;
}
.sub{
  margin-top:8px;
  opacity:.7;
}
button{
  margin-top:22px;
  width:100%;
  border:0;
  border-radius:18px;
  padding:15px;
  font-size:18px;
  font-weight:800;
  color:#07151b;
  background:#ffe477;
}
#over{display:none}
.tip{
  margin-top:14px;
  font-size:13px;
  opacity:.65;
  line-height:1.7;
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="hud">
  <div class="box">
    <div class="small">距离</div>
    <div class="big"><span id="distance">0</span> m</div>
  </div>

  <div class="box">
    🪙 <span id="coins">0</span>
  </div>
</div>

<div id="intro">你们好，我是臭臭鼠</div>

<div id="start">
  <div class="card">
    <div class="title">孤岛臭臭鼠</div>
    <div class="sub">海岛极速逃跑</div>
    <button id="startBtn">开始游戏</button>
    <div class="tip">
      左右滑动：换道<br>
      上滑：跳跃
    </div>
  </div>
</div>

<div id="over">
  <div class="card">
    <div class="title">游戏结束</div>
    <div class="sub">
      跑了 <b id="finalDistance">0</b> 米<br>
      获得 🪙 <b id="finalCoins">0</b>
    </div>
    <button id="restartBtn">再跑一次</button>
  </div>
</div>

<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let W=0,H=0,DPR=1;

function resize(){
  DPR=Math.min(devicePixelRatio||1,2);
  W=innerWidth;
  H=innerHeight;

  canvas.width=W*DPR;
  canvas.height=H*DPR;
  ctx.setTransform(DPR,0,0,DPR,0,0);
}
addEventListener("resize",resize);
resize();

const $=id=>document.getElementById(id);

let game={
  state:"menu",

  distance:0,
  coins:0,

  lane:1,
  targetLane:1,

  jumpY:0,
  jumpV:0,

  speed:0.075,

  objects:[],
  particles:[],

  spawnTimer:0,

  time:0,

  introTime:0,
  introPlayed:false,

  shake:0,

  best:Number(localStorage.getItem("chouchoushuBest")||0)
};

/* =========================
   基础工具
========================= */

function clamp(v,a,b){
  return Math.max(a,Math.min(b,v));
}

function lerp(a,b,t){
  return a+(b-a)*t;
}

function rand(a,b){
  return a+Math.random()*(b-a);
}

function choose(arr){
  return arr[(Math.random()*arr.length)|0];
}

/* =========================
   透视系统
========================= */

const horizonRatio=.39;

function horizon(){
  return H*horizonRatio;
}

/*
 z:
 1 = 很远
 0 = 玩家脚下
*/

function perspective(z){
  return Math.pow(clamp(1-z,0,1),1.7);
}

function roadY(z){
  return lerp(horizon(),H*.94,perspective(z));
}

function roadWidth(z){
  return lerp(W*.10,W*.82,perspective(z));
}

function laneX(lane,z){
  const center=W/2;
  const width=roadWidth(z);

  return center+(lane-1)*width*.285;
}

/* =========================
   天空
========================= */

function drawSky(){

  let sky=ctx.createLinearGradient(
    0,0,0,H
  );

  sky.addColorStop(0,"#76cfe1");
  sky.addColorStop(.48,"#b9e8e7");
  sky.addColorStop(1,"#4b9e9c");

  ctx.fillStyle=sky;
  ctx.fillRect(0,0,W,H);

  /* 太阳 */
  ctx.beginPath();
  ctx.arc(W*.78,H*.17,42,0,Math.PI*2);
  ctx.fillStyle="rgba(255,239,171,.72)";
  ctx.fill();

  /* 云 */
  drawCloud(W*.18,H*.16,1);
  drawCloud(W*.58,H*.11,.75);
  drawCloud(W*.88,H*.28,.65);
}

function drawCloud(x,y,s){

  ctx.save();
  ctx.globalAlpha=.38;
  ctx.fillStyle="#fff";

  ctx.beginPath();
  ctx.arc(x,y,25*s,0,Math.PI*2);
  ctx.arc(x+25*s,y+5*s,19*s,0,Math.PI*2);
  ctx.arc(x-25*s,y+8*s,18*s,0,Math.PI*2);
  ctx.fill();

  ctx.restore();
}

/* =========================
   海
========================= */

function drawSea(){

  ctx.fillStyle="#2b9aa0";

  ctx.beginPath();
  ctx.moveTo(0,horizon());
  ctx.lineTo(W,horizon());

  for(let x=W;x>=0;x-=10){

    const wave=
      Math.sin(x*.025+game.time*.00025)*3+
      Math.sin(x*.008+game.time*.0001)*4;

    ctx.lineTo(x,horizon()+18+wave);
  }

  ctx.closePath();
  ctx.fill();

  /* 海浪 */
  ctx.globalAlpha=.28;

  for(let i=0;i<8;i++){

    const y=horizon()+25+i*17;

    ctx.beginPath();

    for(let x=0;x<W;x+=18){
      ctx.lineTo(
        x,
        y+Math.sin(x*.04+game.time*.001+i)*3
      );
    }

    ctx.strokeStyle="#d9ffff";
    ctx.lineWidth=1;
    ctx.stroke();
  }

  ctx.globalAlpha=1;
}

/* =========================
   远处岛屿
========================= */

function drawIsland(){

  const y=horizon()+18;

  ctx.fillStyle="#315f54";

  ctx.beginPath();

  ctx.moveTo(0,y+8);

  ctx.quadraticCurveTo(
    W*.17,y-35,
    W*.31,y+5
  );

  ctx.quadraticCurveTo(
    W*.48,y-50,
    W*.64,y+4
  );

  ctx.quadraticCurveTo(
    W*.83,y-32,
    W,y+5
  );

  ctx.lineTo(W,y+65);
  ctx.lineTo(0,y+65);

  ctx.closePath();
  ctx.fill();

  /* 远处树 */
  for(let i=0;i<9;i++){

    const x=(i+.5)*W/9;

    const ty=y+rand(-25,5);

    ctx.fillStyle="#244d44";
    ctx.fillRect(x-3,ty,6,25);

    ctx.beginPath();
    ctx.arc(x,ty-8,18,0,Math.PI*2);
    ctx.fill();
  }
}

/* =========================
   道路
========================= */

function drawRoad(){

  const hy=horizon();

  /* 草地 */
  ctx.fillStyle="#5c9b65";

  ctx.beginPath();
  ctx.moveTo(0,hy);
  ctx.lineTo(W,hy);
  ctx.lineTo(W,H);
  ctx.lineTo(0,H);
  ctx.closePath();
  ctx.fill();

  /* 道路主体 */

  const farW=roadWidth(1);
  const nearW=roadWidth(0);

  ctx.beginPath();

  ctx.moveTo(W/2-farW/2,hy);
  ctx.lineTo(W/2+farW/2,hy);

  ctx.lineTo(W/2+nearW/2,H);
  ctx.lineTo(W/2-nearW/2,H);

  ctx.closePath();

  const roadGrad=ctx.createLinearGradient(
    0,hy,0,H
  );

  roadGrad.addColorStop(0,"#687a78");
  roadGrad.addColorStop(.5,"#52625f");
  roadGrad.addColorStop(1,"#394744");

  ctx.fillStyle=roadGrad;
  ctx.fill();

  /* 道路边缘 */

  ctx.strokeStyle="#e5d9a1";
  ctx.lineWidth=5;

  ctx.beginPath();
  ctx.moveTo(W/2-farW/2,hy);
  ctx.lineTo(W/2-nearW/2,H);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(W/2+farW/2,hy);
  ctx.lineTo(W/2+nearW/2,H);
  ctx.stroke();

  /* 三条路线分隔 */

  for(let lane=0;lane<2;lane++){

    for(let i=0;i<18;i++){

      const z=1-i/18;

      const z2=z-.035;

      if(z2<0)continue;

      const x1=
        W/2+
        (lane===0?-1:1)*roadWidth(z)*.145;

      const x2=
        W/2+
        (lane===0?-1:1)*roadWidth(z2)*.145;

      const y1=roadY(z);
      const y2=roadY(z2);

      ctx.strokeStyle="rgba(240,239,205,.55)";
      ctx.lineWidth=Math.max(1,4*perspective(z));

      ctx.beginPath();
      ctx.moveTo(x1,y1);
      ctx.lineTo(x2,y2);
      ctx.stroke();
    }
  }

  /* 路面砖块 */

  for(let i=0;i<28;i++){

    const z=(i/28);

    const y=roadY(z);

    const w=roadWidth(z);

    ctx.strokeStyle="rgba(255,255,255,.045)";
    ctx.lineWidth=1;

    ctx.beginPath();
    ctx.moveTo(W/2-w/2,y);
    ctx.lineTo(W/2+w/2,y);
    ctx.stroke();
  }
}

/* =========================
   路边植物
========================= */

function drawPalm(x,y,s){

  ctx.save();
  ctx.translate(x,y);
  ctx.scale(s,s);

  ctx.strokeStyle="#704b2d";
  ctx.lineWidth=6;
  ctx.lineCap="round";

  ctx.beginPath();
  ctx.moveTo(0,0);
  ctx.quadraticCurveTo(
    -5,-35,
    4,-70
  );
  ctx.stroke();

  ctx.fillStyle="#267449";

  for(let i=0;i<7;i++){

    const a=
      -1.4+i*.45+
      Math.sin(game.time*.001+i)*.03;

    ctx.save();
    ctx.rotate(a);

    ctx.beginPath();
    ctx.ellipse(
      0,-72,
      8,32,
      0,0,Math.PI*2
    );
    ctx.fill();

    ctx.restore();
  }

  ctx.restore();
}

function drawEnvironment(){

  for(let i=0;i<8;i++){

    const z=(i+.5)/8;

    const y=roadY(z);

    const scale=.15+.7*perspective(z);

    if(i%2===0){
      drawPalm(
        roadWidth(z)/2+W/2+20,
        y,
        scale
      );
    }else{
      drawPalm(
        W/2-roadWidth(z)/2-20,
        y,
        scale
      );
    }
  }
}

/* =========================
   老鼠
========================= */

/*
  这里才是重点：
  老鼠本身有真实跑步动画。
  身体会上下起伏，双腿交替，
  尾巴摆动，耳朵也会轻微运动。
*/

function drawMouse(x,y,scale,mode="run"){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(scale,scale);

  let t=game.time*.012;

  let running=mode==="run";

  let step=running?
    Math.sin(t)*11:
    0;

  let bounce=running?
    Math.abs(Math.sin(t))*.035:
    0;

  ctx.translate(0,-bounce*100);

  /* 阴影 */

  ctx.save();

  ctx.scale(1,.28);

  ctx.beginPath();
  ctx.ellipse(
    0,8,
    37,13,
    0,0,Math.PI*2
  );

  ctx.fillStyle="rgba(0,0,0,.3)";
  ctx.fill();

  ctx.restore();

  /* 尾巴 */

  ctx.strokeStyle="#9d6950";
  ctx.lineWidth=7;
  ctx.lineCap="round";

  ctx.beginPath();

  ctx.moveTo(27,5);

  ctx.bezierCurveTo(
    62,-8,
    68+Math.sin(t)*8,-35,
    89, -17
  );

  ctx.stroke();

  /* 后腿 */

  ctx.strokeStyle="#734837";
  ctx.lineWidth=9;

  ctx.beginPath();
  ctx.moveTo(-11,24);
  ctx.lineTo(-18-step*.55,46);
  ctx.lineTo(-4-step,56);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(10,24);
  ctx.lineTo(17+step*.55,46);
  ctx.lineTo(29+step,56);
  ctx.stroke();

  /* 身体 */

  ctx.fillStyle="#a86c4e";

  ctx.beginPath();
  ctx.ellipse(
    0,5,
    33,39,
    0,0,Math.PI*2
  );

  ctx.fill();

  /* 肚子 */

  ctx.fillStyle="#d99b76";

  ctx.beginPath();
  ctx.ellipse(
    5,14,
    20,25,
    0,0,Math.PI*2
  );

  ctx.fill();

  /* 头 */

  ctx.fillStyle="#b87958";

  ctx.beginPath();
  ctx.ellipse(
    -4,-35,
    32,29,
    0,0,Math.PI*2
  );

  ctx.fill();

  /* 耳朵 */

  ctx.fillStyle="#8d5844";

  ctx.beginPath();
  ctx.arc(-28,-57,15,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(20,-58,15,0,Math.PI*2);
  ctx.fill();

  ctx.fillStyle="#e5a38d";

  ctx.beginPath();
  ctx.arc(-28,-57,8,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(20,-58,8,0,Math.PI*2);
  ctx.fill();

  /* 眼睛 */

  ctx.fillStyle="#171313";

  ctx.beginPath();
  ctx.arc(-14,-39,4,0,Math.PI*2);
  ctx.fill();

  ctx.beginPath();
  ctx.arc(9,-39,4,0,Math.PI*2);
  ctx.fill();

  /* 鼻子 */

  ctx.fillStyle="#321d1b";

  ctx.beginPath();
  ctx.arc(-2,-25,5,0,Math.PI*2);
  ctx.fill();

  /* 嘴 */

  ctx.strokeStyle="#542c29";
  ctx.lineWidth=2;

  ctx.beginPath();
  ctx.arc(-2,-23,9,.15,1.1);
  ctx.stroke();

  /* 前腿 */

  ctx.strokeStyle="#744635";
  ctx.lineWidth=8;

  ctx.beginPath();
  ctx.moveTo(-19,18);
  ctx.lineTo(-28-step,42);
  ctx.lineTo(-17-step,51);
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(17,18);
  ctx.lineTo(28+step,42);
  ctx.lineTo(39+step,51);
  ctx.stroke();

  ctx.restore();
}

/* =========================
   物体
========================= */

function spawn(type,lane,z=1.02){

  game.objects.push({
    type,
    lane,
    z,
    collected:false,
    rot:Math.random()*6.28
  });
}

function spawnPattern(){

  const safe=Math.floor(Math.random()*3);

  const r=Math.random();

  /* 单个障碍 */

  if(r<.24){

    spawn(
      choose(["rock","log","barrier"]),
      choose([0,1,2]),
      1.05
    );

  }

  /* 双障碍 */

  else if(r<.43){

    const lanes=[0,1,2];

    lanes.splice(safe,1);

    spawn(
      choose(["rock","log","barrier"]),
      lanes[0],
      1.05
    );

    spawn(
      choose(["rock","log"]),
      lanes[1],
      1.18
    );
  }

  /* 金币路线 */

  else if(r<.63){

    const lane=choose([0,1,2]);

    for(let i=0;i<5;i++){

      spawn(
        "coin",
        lane,
        1.05+i*.08
      );
    }
  }

  /* 道具 */

  else if(r<.78){

    spawn(
      choose(["boost","magnet","shield"]),
      safe,
      1.08
    );

    for(let i=0;i<4;i++){

      spawn(
        "coin",
        safe,
        1.22+i*.08
      );
    }
  }

  /* 宝箱 */

  else{

    spawn("chest",safe,1.05);

    for(let i=0;i<5;i++){

      spawn(
        "coin",
        safe,
        1.18+i*.07
      );
    }
  }
}

/* =========================
   物体绘制
========================= */

function drawObject(o){

  const p=perspective(o.z);

  const x=laneX(o.lane,o.z);
  const y=roadY(o.z);

  const s=.18+.95*p;

  ctx.save();

  ctx.translate(x,y);

  if(o.type==="coin"){

    ctx.rotate(
      Math.sin(game.time*.006+o.rot)*.35
    );

    ctx.fillStyle="#ffd84a";

    ctx.beginPath();
    ctx.ellipse(
      0,
      -25*s,
      13*s,
      17*s,
      0,
      0,
      Math.PI*2
    );
    ctx.fill();

    ctx.strokeStyle="#fff2a2";
    ctx.lineWidth=3*s;

    ctx.stroke();

  }

  else if(o.type==="rock"){

    ctx.fillStyle="#655c54";

    ctx.beginPath();

    ctx.moveTo(-25*s,0);
    ctx.lineTo(-18*s,-35*s);
    ctx.lineTo(8*s,-43*s);
    ctx.lineTo(28*s,-15*s);
    ctx.lineTo(22*s,0);

    ctx.closePath();
    ctx.fill();

    ctx.fillStyle="#81766b";

    ctx.beginPath();
    ctx.moveTo(-18*s,-35*s);
    ctx.lineTo(8*s,-43*s);
    ctx.lineTo(0,-28*s);
    ctx.closePath();
    ctx.fill();
  }

  else if(o.type==="log"){

    ctx.fillStyle="#784b2f";

    ctx.rotate(.05);

    ctx.fillRect(
      -32*s,
      -27*s,
      64*s,
      25*s
    );

    ctx.fillStyle="#b67a4e";

    ctx.beginPath();
    ctx.arc(
      31*s,
      -15*s,
      12*s,
      0,
      Math.PI*2
    );
    ctx.fill();

  }

  else if(o.type==="barrier"){

    ctx.fillStyle="#d35a42";

    ctx.fillRect(
      -32*s,
      -34*s,
      64*s,
      28*s
    );

    ctx.fillStyle="#f4e4bd";

    for(let i=-2;i<3;i++){

      ctx.save();
      ctx.translate(i*16*s,-20*s);
      ctx.rotate(-.5);

      ctx.fillRect(
        -4*s,
        -14*s,
        8*s,
        28*s
      );

      ctx.restore();
    }
  }

  else if(o.type==="boost"){

    drawPower(
      "⚡",
      "#ffca4a",
      s
    );
  }

  else if(o.type==="magnet"){

    drawPower(
      "🧲",
      "#e85a7a",
      s
    );
  }

  else if(o.type==="shield"){

    drawPower(
      "🛡️",
      "#58c8ff",
      s
    );
  }

  else if(o.type==="chest"){

    ctx.fillStyle="#a96532";

    ctx.fillRect(
      -30*s,
      -40*s,
      60*s,
      40*s
    );

    ctx.fillStyle="#ffd34c";

    ctx.fillRect(
      -6*s,
      -40*s,
      12*s,
      40*s
    );

    ctx.beginPath();

    ctx.arc(
      0,
      -40*s,
      30*s,
      Math.PI,
      Math.PI*2
    );

    ctx.fillStyle="#c27b3b";
    ctx.fill();
  }

  ctx.restore();
}

function drawPower(icon,color,s){

  ctx.shadowColor=color;
  ctx.shadowBlur=20*s;

  ctx.fillStyle="rgba(255,255,255,.18)";

  ctx.beginPath();
  ctx.arc(
    0,
    -25*s,
    27*s,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.shadowBlur=0;

  ctx.font=`${34*s}px sans-serif`;
  ctx.textAlign="center";
  ctx.textBaseline="middle";

  ctx.fillText(
    icon,
    0,
    -25*s
  );
}

/* =========================
   粒子
========================= */

function particle(x,y,color){

  game.particles.push({
    x,y,
    vx:rand(-2,2),
    vy:rand(-4,-1),
    life:1,
    color
  });
}

function updateParticles(dt){

  for(let i=game.particles.length-1;i>=0;i--){

    const p=game.particles[i];

    p.x+=p.vx;
    p.y+=p.vy;

    p.vy+=.15;

    p.life-=dt*2;

    if(p.life<=0){
      game.particles.splice(i,1);
    }
  }
}

function drawParticles(){

  for(const p of game.particles){

    ctx.globalAlpha=p.life;
    ctx.fillStyle=p.color;

    ctx.beginPath();
    ctx.arc(
      p.x,
      p.y,
      3,
      0,
      Math.PI*2
    );

    ctx.fill();
  }

  ctx.globalAlpha=1;
}

/* =========================
   开始
========================= */

function startGame(){

  game.state="intro";

  game.distance=0;
  game.coins=0;

  game.lane=1;
  game.targetLane=1;

  game.jumpY=0;
  game.jumpV=0;

  game.speed=.075;

  game.objects=[];
  game.particles=[];

  game.spawnTimer=.8;

  game.introTime=0;

  $("start").style.display="none";
  $("over").style.display="none";

  $("intro").style.opacity=0;
}

/* =========================
   正式进入跑步
========================= */

function beginRun(){

  game.state="run";

  $("intro").style.opacity=0;
}

/* =========================
   游戏结束
========================= */

function gameOver(){

  if(game.state==="over")return;

  game.state="over";

  game.best=Math.max(
    game.best,
    Math.floor(game.distance)
  );

  localStorage.setItem(
    "chouchoushuBest",
    game.best
  );

  $("finalDistance").textContent=
    Math.floor(game.distance);

  $("finalCoins").textContent=
    game.coins;

  $("over").style.display="flex";
}

/* =========================
   开场动画
========================= */

function updateIntro(dt){

  game.introTime+=dt;

  const t=game.introTime;

  const intro=$("intro");

  if(t<.7){

    intro.style.opacity=
      clamp(t/.7,0,1);

  }
  else if(t<2.5){

    intro.style.opacity=1;

  }
  else if(t<3.2){

    intro.style.opacity=
      1-(t-2.5)/.7;

  }

  if(t>=3.4){

    beginRun();

  }
}

/* =========================
   更新游戏
========================= */

function update(dt){

  if(game.state==="intro"){

    updateIntro(dt);
    return;
  }

  if(game.state!=="run"){
    return;
  }

  /*
    距离真正持续增加。
    老鼠的位置固定在玩家附近，
    但道路和物体根据 z 不断向玩家移动。
  */

  let speed=game.speed;

  /* 速度成长 */

  if(game.distance<100){

    speed=.060;

  }else if(game.distance<300){

    speed=.072;

  }else if(game.distance<600){

    speed=.084;

  }else if(game.distance<1000){

    speed=.098;

  }else{

    speed=.112;

  }

  game.speed=lerp(
    game.speed,
    speed,
    dt*2
  );

  game.distance+=game.speed*dt*60;

  /* 左右换道 */

  game.lane=lerp(
    game.lane,
    game.targetLane,
    dt*12
  );

  /* 跳跃 */

  if(game.jumpY>0 || game.jumpV>0){

    game.jumpV-=.65*dt*60;

    game.jumpY+=game.jumpV*dt*60;

    if(game.jumpY<=0){

      game.jumpY=0;
      game.jumpV=0;
    }
  }

  /* 生成 */

  game.spawnTimer-=dt;

  if(game.spawnTimer<=0){

    spawnPattern();

    game.spawnTimer=
      Math.max(
        .62,
        1.15-game.speed*3
      );
  }

  /* 物体向玩家移动 */

  for(let i=game.objects.length-1;i>=0;i--){

    const o=game.objects[i];

    /*
      z越小代表越靠近玩家
    */

    o.z-=game.speed*dt*1.35;

    const x=laneX(o.lane,o.z);
    const y=roadY(o.z);

    /*
      收集金币 / 道具
    */

    if(
      !o.collected &&
      o.z<.115 &&
      o.z>-.08 &&
      Math.abs(
        o.lane-game.lane
      )<.32
    ){

      if(o.type==="coin"){

        o.collected=true;
        game.coins++;

        for(let k=0;k<6;k++){

          particle(
            x,
            y-20,
            "#ffe05b"
          );
        }
      }

      else if(
        ["boost","magnet","shield","chest"]
        .includes(o.type)
      ){

        o.collected=true;

        for(let k=0;k<10;k++){

          particle(
            x,
            y-20,
            "#8eeeff"
          );
        }
      }

      else if(
        ["rock","log","barrier"]
        .includes(o.type)
      ){

        /*
          跳跃状态下可以躲过
        */

        if(game.jumpY<25){

          game.shake=12;
          gameOver();

        }
      }
    }

    if(o.z<-.18){

      game.objects.splice(i,1);
    }
  }

  updateParticles(dt);

  if(game.shake>0){

    game.shake-=dt*30;

    if(game.shake<0)
      game.shake=0;
  }

  $("distance").textContent=
    Math.floor(game.distance);

  $("coins").textContent=
    game.coins;
}

/* =========================
   绘制
========================= */

function draw(){

  ctx.clearRect(0,0,W,H);

  ctx.save();

  if(game.shake>0){

    ctx.translate(
      rand(-game.shake,game.shake),
      rand(-game.shake,game.shake)
    );
  }

  drawSky();
  drawSea();
  drawIsland();
  drawEnvironment();
  drawRoad();

  /*
    按远近绘制物体，
    越远越先画。
  */

  const sorted=
    [...game.objects]
    .sort((a,b)=>b.z-a.z);

  for(const o of sorted){

    if(o.z>0){

      drawObject(o);
    }
  }

  /*
    老鼠始终处在玩家位置。
    它自身不断跑步，
    而道路/障碍向后移动。
  */

  if(
    game.state==="run" ||
    game.state==="intro"
  ){

    let mouseX;
    let mouseY;

    if(game.state==="intro"){

      const t=game.introTime;

      /*
        开场：
        先从远处跑过来，
        然后停下来挥手。
      */

      const p=
        clamp(t/1.2,0,1);

      mouseX=
        W/2;

      mouseY=
        lerp(
          H*.72,
          H*.67,
          p
        );

      let s=
        lerp(.35,.9,p);

      drawMouse(
        mouseX,
        mouseY,
        s,
        t<1.25?"run":"idle"
      );

    }else{

      mouseX=
        laneX(
          game.lane,
          .045
        );

      mouseY=
        roadY(.045)-game.jumpY;

      drawMouse(
        mouseX,
        mouseY,
        .82,
        "run"
      );
    }
  }

  drawParticles();

  ctx.restore();
}

/* =========================
   主循环
========================= */

let last=performance.now();

function loop(now){

  let dt=
    Math.min(
      .033,
      (now-last)/1000
    );

  last=now;

  game.time+=dt*1000;

  update(dt);
  draw();

  requestAnimationFrame(loop);
}

requestAnimationFrame(loop);

/* =========================
   手机触摸控制
========================= */

let touchX=0;
let touchY=0;

canvas.addEventListener(
  "touchstart",
  e=>{

    const t=e.touches[0];

    touchX=t.clientX;
    touchY=t.clientY;

  },
  {passive:true}
);

canvas.addEventListener(
  "touchend",
  e=>{

    if(game.state!=="run")
      return;

    const t=e.changedTouches[0];

    const dx=t.clientX-touchX;
    const dy=t.clientY-touchY;

    /*
      横向滑动
    */

    if(
      Math.abs(dx)>
      Math.abs(dy)
    ){

      if(Math.abs(dx)>25){

        if(dx>0){

          game.targetLane=
            Math.min(
              2,
              game.targetLane+1
            );

        }else{

          game.targetLane=
            Math.max(
              0,
              game.targetLane-1
            );
        }
      }

    }

    /*
      上滑跳跃
    */

    else{

      if(
        dy<-35 &&
        game.jumpY<=0
      ){

        game.jumpV=12;
        game.jumpY=1;
      }
    }

  },
  {passive:true}
);

/* =========================
   键盘
========================= */

addEventListener(
  "keydown",
  e=>{

    if(game.state!=="run")
      return;

    if(
      e.key==="ArrowLeft" ||
      e.key==="a"
    ){

      game.targetLane=
        Math.max(
          0,
          game.targetLane-1
        );
    }

    if(
      e.key==="ArrowRight" ||
      e.key==="d"
    ){

      game.targetLane=
        Math.min(
          2,
          game.targetLane+1
        );
    }

    if(
      e.key==="ArrowUp" ||
      e.key==="w" ||
      e.key===" "
    ){

      if(game.jumpY<=0){

        game.jumpV=12;
        game.jumpY=1;
      }
    }

  }
);

/* =========================
   按钮
========================= */

$("startBtn").onclick=
  startGame;

$("restartBtn").onclick=
  startGame;

</script>

</body>
</html>
