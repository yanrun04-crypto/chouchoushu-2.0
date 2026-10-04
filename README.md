<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">

<title>孤岛臭臭鼠</title>

<style>
*{
  box-sizing:border-box;
  -webkit-tap-highlight-color:transparent;
  user-select:none;
}

html,body{
  margin:0;
  width:100%;
  height:100%;
  overflow:hidden;
  background:#73c9e7;
  font-family:-apple-system,BlinkMacSystemFont,
  "PingFang SC","Microsoft YaHei",sans-serif;
}

body{
  touch-action:none;
}

canvas{
  position:fixed;
  inset:0;
  width:100%;
  height:100%;
}

#hud{
  position:fixed;
  z-index:20;
  top:0;
  left:0;
  right:0;
  padding:calc(10px + env(safe-area-inset-top)) 12px 0;
  pointer-events:none;
  color:white;
  text-shadow:0 2px 5px rgba(0,0,0,.35);
}

.hudRow{
  display:flex;
  justify-content:space-between;
  align-items:flex-start;
}

.hudBox{
  padding:7px 12px;
  border-radius:16px;
  background:rgba(20,65,80,.30);
  border:1px solid rgba(255,255,255,.25);
  backdrop-filter:blur(8px);
}

.small{
  font-size:11px;
  opacity:.82;
}

.big{
  font-size:20px;
  font-weight:900;
}

#statusBox{
  margin-top:7px;
  width:max-content;
}

#power{
  width:120px;
  height:5px;
  margin-top:5px;
  border-radius:10px;
  background:rgba(255,255,255,.22);
  overflow:hidden;
}

#powerFill{
  width:0;
  height:100%;
  background:#ffe05c;
  border-radius:10px;
}

.screen{
  position:fixed;
  inset:0;
  z-index:50;
  display:flex;
  align-items:center;
  justify-content:center;
  background:
    linear-gradient(
      #55bce8 0%,
      #b8e7ee 57%,
      #e5e8c8 100%
    );
}

.panel{
  width:min(88vw,420px);
  padding:28px 22px 23px;
  border-radius:28px;
  background:rgba(255,255,255,.95);
  box-shadow:0 25px 70px rgba(0,0,0,.22);
  text-align:center;
}

h1{
  margin:0;
  color:#293c4c;
  font-size:32px;
}

.sub{
  margin:7px 0 18px;
  color:#73828d;
  font-size:13px;
}

button{
  width:100%;
  border:0;
  border-radius:17px;
  padding:15px;
  background:linear-gradient(#55cdf5,#178dcc);
  color:white;
  font-size:18px;
  font-weight:900;
  box-shadow:0 5px 0 #0d6a98;
}

button:active{
  transform:translateY(4px);
  box-shadow:0 1px 0 #0d6a98;
}

.tip{
  margin-top:15px;
  color:#89949c;
  font-size:12px;
  line-height:1.8;
}

.hidden{
  display:none!important;
}

.mouseIcon{
  width:105px;
  height:105px;
  margin:0 auto 15px;
  border-radius:50%;
  background:#a6785f;
  position:relative;
  box-shadow:
    inset -14px -15px rgba(0,0,0,.10),
    0 12px 22px rgba(0,0,0,.16);
}

.mouseIcon:before,
.mouseIcon:after{
  content:"";
  position:absolute;
  width:40px;
  height:40px;
  border-radius:50%;
  background:#b98a6d;
  top:-9px;
}

.mouseIcon:before{
  left:6px;
}

.mouseIcon:after{
  right:6px;
}

.eye{
  position:absolute;
  width:8px;
  height:12px;
  border-radius:50%;
  background:#172027;
  top:43px;
}

.eye.l{
  left:31px;
}

.eye.r{
  right:31px;
}

.nose{
  position:absolute;
  left:50%;
  top:60px;
  transform:translateX(-50%);
  width:12px;
  height:9px;
  border-radius:50%;
  background:#e59ca7;
}

#introText{
  position:fixed;
  z-index:40;
  left:50%;
  bottom:18%;
  transform:translateX(-50%);
  color:white;
  font-size:22px;
  font-weight:900;
  white-space:nowrap;
  text-shadow:0 3px 9px rgba(20,55,70,.8);
  opacity:0;
  pointer-events:none;
}

#skip{
  display:none;
  position:fixed;
  right:12px;
  top:calc(12px + env(safe-area-inset-top));
  z-index:45;
  padding:7px 11px;
  border-radius:15px;
  color:white;
  background:rgba(0,0,0,.25);
  font-size:12px;
}

#toast{
  position:fixed;
  left:50%;
  top:24%;
  transform:translate(-50%,-50%);
  z-index:45;
  color:white;
  font-size:27px;
  font-weight:900;
  text-shadow:0 3px 8px #234;
  opacity:0;
  pointer-events:none;
}

#hint{
  position:fixed;
  left:50%;
  bottom:7%;
  transform:translateX(-50%);
  z-index:15;
  color:white;
  font-size:13px;
  font-weight:700;
  opacity:0;
  pointer-events:none;
  text-shadow:0 2px 5px rgba(0,0,0,.45);
}
</style>
</head>

<body>

<canvas id="game"></canvas>

<div id="hud">

  <div class="hudRow">

    <div class="hudBox">
      <div class="small">距离</div>
      <div class="big">
        <span id="distance">0</span> m
      </div>
    </div>

    <div class="hudBox" style="text-align:right">
      <div class="small">金币</div>
      <div class="big">
        🪙 <span id="coins">0</span>
      </div>
    </div>

  </div>

  <div id="statusBox" class="hudBox">

    <div class="small">状态</div>

    <div id="status">正常</div>

    <div id="power">
      <div id="powerFill"></div>
    </div>

  </div>

</div>

<div id="introText">你们好，我是臭臭鼠</div>

<div id="skip">跳过 ›</div>

<div id="toast"></div>

<div id="hint">左右滑动换道　↑ 上滑跳跃</div>


<!-- 开始界面 -->

<div id="startScreen" class="screen">

  <div class="panel">

    <div class="mouseIcon">
      <i class="eye l"></i>
      <i class="eye r"></i>
      <i class="nose"></i>
    </div>

    <h1>孤岛臭臭鼠</h1>

    <div class="sub">
      原创海岛 · 3D感无限跑酷
    </div>

    <button id="startBtn">
      开始奔跑
    </button>

    <div class="tip">
      左右滑动：换道<br>
      上滑：跳跃　下滑：快速落地<br>
      收集金币，躲开障碍，跑得越远越好
    </div>

  </div>

</div>


<!-- 游戏结束 -->

<div id="gameOver" class="screen hidden">

  <div class="panel">

    <h1>跑到这里啦！</h1>

    <div style="margin-top:20px;color:#7d8992">
      本次距离
    </div>

    <div id="finalDistance"
    style="font-size:45px;font-weight:1000;color:#263b4b">
      0m
    </div>

    <div style="margin-top:8px;color:#7d8992">
      金币：<span id="finalCoins">0</span>
    </div>

    <div style="margin-top:6px;color:#7d8992">
      最高：<span id="bestDistance">0</span>m
    </div>

    <button id="restartBtn" style="margin-top:18px">
      再跑一次
    </button>

  </div>

</div>


<script>

const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");

let W=innerWidth;
let H=innerHeight;
let DPR=Math.min(devicePixelRatio||1,2);

function resize(){

  W=innerWidth;
  H=innerHeight;

  canvas.width=W*DPR;
  canvas.height=H*DPR;

  canvas.style.width=W+"px";
  canvas.style.height=H+"px";

  ctx.setTransform(DPR,0,0,DPR,0,0);
}

addEventListener("resize",resize);
resize();

const $=id=>document.getElementById(id);

function clamp(v,a,b){
  return Math.max(a,Math.min(b,v));
}

function lerp(a,b,t){
  return a+(b-a)*t;
}

function rand(a,b){
  return a+Math.random()*(b-a);
}

function choose(a){
  return a[Math.floor(Math.random()*a.length)];
}


/* =====================================================
   游戏状态
===================================================== */

const G={

  mode:"menu",

  time:0,
  last:0,

  distance:0,
  coins:0,

  best:Number(
    localStorage.getItem("chouchou_best")||0
  ),

  lane:1,
  targetLane:1,

  jump:0,
  jumpVelocity:0,

  speed:.055,

  objects:[],

  particles:[],

  spawnTimer:1.8,

  shield:0,
  magnet:0,
  boost:0,

  shake:0,

  introTime:0,

  worldOffset:0,

  lastPattern:0
};


/* =====================================================
   透视
===================================================== */

function depth(z){

  return Math.pow(
    clamp(z,0,1),
    1.55
  );
}


function roadY(z){

  return lerp(
    H*.37,
    H*1.08,
    depth(z)
  );
}


function roadWidth(z){

  return lerp(
    W*.095,
    W*.92,
    depth(z)
  );
}


function curveAt(z){

  let a=
    Math.sin(
      (G.distance+z*850)*.00135
    )*W*.055;

  let b=
    Math.sin(
      (G.distance+z*1500)*.00042
    )*W*.028;

  return a+b;
}


function laneX(lane,z){

  return (
    W/2+
    curveAt(z)+
    (lane-1)*roadWidth(z)*.235
  );
}


/* =====================================================
   背景
===================================================== */

function drawSky(){

  let g=ctx.createLinearGradient(
    0,0,0,H
  );

  g.addColorStop(0,"#48b8e6");
  g.addColorStop(.46,"#aee2ed");
  g.addColorStop(1,"#e5e9cf");

  ctx.fillStyle=g;
  ctx.fillRect(0,0,W,H);

  /* 云 */

  for(let i=0;i<6;i++){

    let x=
      ((i*280-G.distance*.012)
      %(W+360))-180;

    let y=
      50+(i%3)*70;

    ctx.fillStyle=
      "rgba(255,255,255,.46)";

    ctx.beginPath();

    ctx.arc(x,y,23,0,Math.PI*2);
    ctx.arc(x+30,y-8,31,0,Math.PI*2);
    ctx.arc(x+62,y,22,0,Math.PI*2);

    ctx.fill();
  }
}


function drawSea(){

  let hy=H*.37;

  let g=ctx.createLinearGradient(
    0,hy,0,H
  );

  g.addColorStop(0,"#45b4d0");
  g.addColorStop(1,"#207f9e");

  ctx.fillStyle=g;

  ctx.fillRect(
    0,
    hy,
    W,
    H-hy
  );

  for(let i=0;i<18;i++){

    let y=
      hy+14+i*14;

    let offset=
      Math.sin(
        G.time*.0008+i
      )*15;

    ctx.strokeStyle=
      "rgba(255,255,255,.12)";

    ctx.lineWidth=1.5;

    ctx.beginPath();

    for(
      let x=-40;
      x<W+40;
      x+=40
    ){

      let yy=
        y+
        Math.sin(
          x*.035+
          i+
          G.time*.001
        )*2;

      if(x===-40){
        ctx.moveTo(x+offset,yy);
      }else{
        ctx.lineTo(x+offset,yy);
      }

    }

    ctx.stroke();
  }
}


function drawIsland(){

  let hy=H*.37;

  ctx.fillStyle="#4d7467";

  ctx.beginPath();

  ctx.moveTo(0,hy+36);

  for(
    let x=0;
    x<=W;
    x+=22
  ){

    let y=
      hy+
      28+
      Math.sin(x*.012)*14+
      Math.sin(x*.039)*6;

    ctx.lineTo(x,y);
  }

  ctx.lineTo(W,hy+115);
  ctx.lineTo(0,hy+115);

  ctx.fill();

  /* 远处山 */

  ctx.fillStyle="#3e655c";

  ctx.beginPath();

  ctx.moveTo(W*.48,hy+45);

  ctx.lineTo(W*.58,hy-10);
  ctx.lineTo(W*.68,hy+45);

  ctx.lineTo(W*.76,hy+10);
  ctx.lineTo(W*.88,hy+50);

  ctx.lineTo(W,hy+50);

  ctx.lineTo(W,hy+100);

  ctx.lineTo(W*.48,hy+100);

  ctx.closePath();

  ctx.fill();
}


/* =====================================================
   道路
===================================================== */

function drawRoad(){

  let hy=H*.37;
  let bottom=H*1.08;

  let topW=roadWidth(0);
  let bottomW=roadWidth(1);

  let tc=curveAt(0);
  let bc=curveAt(1);

  /* 道路阴影 */

  ctx.fillStyle="rgba(22,46,40,.20)";

  ctx.beginPath();

  ctx.moveTo(
    W/2-topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2+topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2+bottomW/2+bc,
    bottom
  );

  ctx.lineTo(
    W/2-bottomW/2+bc,
    bottom
  );

  ctx.closePath();
  ctx.fill();


  /* 路面 */

  let roadGrad=
    ctx.createLinearGradient(
      0,hy,0,bottom
    );

  roadGrad.addColorStop(0,"#cdb57f");
  roadGrad.addColorStop(.5,"#b99a63");
  roadGrad.addColorStop(1,"#9e7e4e");

  ctx.fillStyle=roadGrad;

  ctx.beginPath();

  ctx.moveTo(
    W/2-topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2+topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2+bottomW/2+bc,
    bottom
  );

  ctx.lineTo(
    W/2-bottomW/2+bc,
    bottom
  );

  ctx.closePath();

  ctx.fill();


  /* 路面横向纹理 */

  for(let i=0;i<34;i++){

    let z1=i/34;
    let z2=(i+1)/34;

    let y1=roadY(z1);
    let y2=roadY(z2);

    let w1=roadWidth(z1);
    let w2=roadWidth(z2);

    ctx.fillStyle=
      i%2===0
      ?"rgba(255,255,255,.025)"
      :"rgba(60,40,20,.035)";

    ctx.beginPath();

    ctx.moveTo(
      W/2-w1/2+curveAt(z1),
      y1
    );

    ctx.lineTo(
      W/2+w1/2+curveAt(z1),
      y1
    );

    ctx.lineTo(
      W/2+w2/2+curveAt(z2),
      y2
    );

    ctx.lineTo(
      W/2-w2/2+curveAt(z2),
      y2
    );

    ctx.closePath();

    ctx.fill();
  }


  /* 道路边缘 */

  ctx.strokeStyle=
    "rgba(245,224,165,.75)";

  ctx.lineWidth=4;

  ctx.beginPath();

  ctx.moveTo(
    W/2-topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2-bottomW/2+bc,
    bottom
  );

  ctx.stroke();

  ctx.beginPath();

  ctx.moveTo(
    W/2+topW/2+tc,
    hy
  );

  ctx.lineTo(
    W/2+bottomW/2+bc,
    bottom
  );

  ctx.stroke();


  /* 三车道分隔线 */

  for(let lane=0;lane<2;lane++){

    let offset=
      (lane-.5)*.47;

    ctx.strokeStyle=
      "rgba(255,241,193,.55)";

    ctx.lineWidth=2;

    ctx.setLineDash([9,17]);

    ctx.beginPath();

    ctx.moveTo(
      W/2+
      offset*roadWidth(0)+
      curveAt(0),
      hy
    );

    ctx.lineTo(
      W/2+
      offset*roadWidth(1)+
      curveAt(1),
      bottom
    );

    ctx.stroke();

    ctx.setLineDash([]);
  }


  /* 路面小石子 */

  for(let i=0;i<32;i++){

    let z=
      (i/32+
      G.worldOffset*.0007+
      i*.013)%1;

    let y=roadY(z);
    let w=roadWidth(z);
    let c=curveAt(z);

    let s=.15+z*1.3;

    ctx.fillStyle=
      i%3===0
      ?"rgba(83,69,48,.30)"
      :"rgba(245,220,165,.28)";

    ctx.beginPath();

    ctx.ellipse(
      W/2+
      ((i%2?-1:1)*w*.37)+c,
      y,
      3*s,
      2*s,
      0,
      0,
      Math.PI*2
    );

    ctx.fill();
  }
}


/* =====================================================
   海岛环境
===================================================== */

function drawPalm(x,y,s){

  ctx.save();

  ctx.translate(x,y);
  ctx.scale(s,s);

  ctx.strokeStyle="#735035";
  ctx.lineWidth=8;
  ctx.lineCap="round";

  ctx.beginPath();

  ctx.moveTo(0,0);

  ctx.quadraticCurveTo(
    -5,-45,
    -2,-100
  );

  ctx.stroke();

  ctx.fillStyle="#30945a";

  for(
    let a=-1.3;
    a<=1.3;
    a+=.43
  ){

    ctx.save();

    ctx.rotate(a);

    ctx.beginPath();

    ctx.ellipse(
      0,-101,
      10,
      37,
      0,
      0,
      Math.PI*2
    );

    ctx.fill();

    ctx.restore();
  }

  ctx.restore();
}


function drawEnvironment(){

  for(let i=0;i<11;i++){

    let z=
      (i/11+
      G.worldOffset*.0005)%1;

    let y=roadY(z);
    let w=roadWidth(z);
    let c=curveAt(z);

    let s=.13+z*1.35;

    let side=
      i%2===0?-1:1;

    drawPalm(
      W/2+
      side*(w/2+34*s)+
      c,
      y+7,
      s
    );

    /* 草丛 */

    if(z>.2){

      ctx.fillStyle="#477b4e";

      let gx=
        W/2+
        side*(w/2+13*s)+
        c;

      ctx.beginPath();

      for(let k=0;k<5;k++){

        ctx.moveTo(
          gx+k*4*side,
          y
        );

        ctx.lineTo(
          gx+(k*4+3)*side,
          y-10*s
        );
      }

      ctx.fill();
    }
  }
}


/* =====================================================
   物体
===================================================== */

function spawn(type,lane,z){

  G.objects.push({
    type,
    lane,
    z,
    phase:Math.random()*Math.PI*2,
    hit:false
  });
}


/*
  新版核心：

  不再连续疯狂刷东西。

  每次只生成一个“事件组”。

  组与组之间必须有明显安全距离。
*/

function spawnPattern(){

  let z=1.08;

  let d=G.distance;

  let r=Math.random();

  /*
    0-100m：
    几乎教学模式
  */

  if(d<100){

    if(Math.random()<.55){

      let lane=
        Math.floor(Math.random()*3);

      spawn("coin",lane,z);

      spawn("coin",lane,z-.075);

      spawn("coin",lane,z-.15);
    }

    return;
  }


  /*
    100-250m：
    单障碍
  */

  if(d<250){

    if(r<.55){

      let lane=
        Math.floor(Math.random()*3);

      spawn(
        choose(["rock","log"]),
        lane,
        z
      );

    }else{

      let lane=
        Math.floor(Math.random()*3);

      spawn("coin",lane,z);
      spawn("coin",lane,z-.09);
      spawn("coin",lane,z-.18);
    }

    return;
  }


  /*
    250-500m：
    单障碍 + 金币路线
  */

  if(d<500){

    let lane=
      Math.floor(Math.random()*3);

    if(r<.58){

      spawn(
        choose([
          "rock",
          "log",
          "barrier"
        ]),
        lane,
        z
      );

      let safe=
        (lane+1+
        Math.floor(Math.random()*2))%3;

      spawn("coin",safe,z-.15);
      spawn("coin",safe,z-.24);
      spawn("coin",safe,z-.33);

    }else{

      spawn("coin",lane,z);
      spawn("coin",lane,z-.09);
      spawn("coin",lane,z-.18);
      spawn("coin",lane,z-.27);
    }

    return;
  }


  /*
    500-800m：
    开始有组合，但依然留安全路线
  */

  if(d<800){

    let safe=
      Math.floor(Math.random()*3);

    if(r<.68){

      for(let lane=0;lane<3;lane++){

        if(lane!==safe){

          spawn(
            choose([
              "rock",
              "log",
              "barrier"
            ]),
            lane,
            z
          );
        }
      }

      spawn("coin",safe,z-.14);
      spawn("coin",safe,z-.23);
      spawn("coin",safe,z-.32);

    }else{

      spawn("boost",safe,z);

      spawn("coin",safe,z-.12);
      spawn("coin",safe,z-.22);
      spawn("coin",safe,z-.32);
    }

    return;
  }


  /*
    800m+：
    难度提升，但不会满屏
  */

  let safe=
    Math.floor(Math.random()*3);

  if(r<.48){

    for(let lane=0;lane<3;lane++){

      if(lane!==safe){

        spawn(
          choose([
            "rock",
            "barrier",
            "log"
          ]),
          lane,
          z
        );
      }
    }

    spawn("coin",safe,z-.13);
    spawn("coin",safe,z-.23);

  }

  else if(r<.68){

    spawn(
      choose([
        "magnet",
        "shield"
      ]),
      safe,
      z
    );

    spawn("coin",safe,z-.13);
    spawn("coin",safe,z-.23);

  }

  else if(r<.78){

    spawn("chest",safe,z);

  }

  else{

    spawn("coin",safe,z);
    spawn("coin",safe,z-.1);
    spawn("coin",safe,z-.2);
  }
}


function updateSpawn(dt){

  G.spawnTimer-=dt;

  if(G.spawnTimer>0)return;

  spawnPattern();

  /*
    这是这版非常重要的地方：

    以前是不到一秒就刷一组。

    现在根据距离逐渐缩短，
    但前期故意留大量空白。
  */

  let interval;

  if(G.distance<100){

    interval=2.25;

  }else if(G.distance<250){

    interval=1.95;

  }else if(G.distance<500){

    interval=1.72;

  }else if(G.distance<800){

    interval=1.52;

  }else{

    interval=1.35;
  }

  G.spawnTimer=
    interval+
    rand(.15,.55);
}


/* =====================================================
   物体绘制
===================================================== */

function drawObject(o){

  if(o.z<0||o.z>1.12)return;

  let x=laneX(o.lane,o.z);
  let y=roadY(o.z);

  let s=.18+o.z*1.75;

  ctx.save();

  ctx.translate(x,y);


  /* 金币 */

  if(o.type==="coin"){

    let r=8+15*s;

    let bob=
      Math.sin(
        G.time*.006+
        o.phase
      )*4*s;

    ctx.translate(
      0,
      -32*s+bob
    );

    ctx.rotate(
      G.time*.003+
      o.phase
    );

    ctx.shadowBlur=12;
    ctx.shadowColor="#ffd52f";

    ctx.fillStyle="#ffd32f";
    ctx.strokeStyle="#fff2a2";
    ctx.lineWidth=2.5;

    ctx.beginPath();

    ctx.ellipse(
      0,0,
      r*.57,
      r,
      0,
      0,
      Math.PI*2
    );

    ctx.fill();
    ctx.stroke();

    ctx.shadowBlur=0;

    ctx.fillStyle="#fff3a0";

    ctx.font=
      `bold ${Math.max(9,r*.75)}px Arial`;

    ctx.textAlign="center";

    ctx.fillText(
      "★",
      0,
      r*.3
    );
  }


  /* 道具 */

  else if(
    o.type==="boost"||
    o.type==="magnet"||
    o.type==="shield"
  ){

    let r=14+22*s;

    let bob=
      Math.sin(
        G.time*.005+
        o.phase
      )*5;

    ctx.translate(
      0,
      -38*s+bob
    );

    let color;

    if(o.type==="boost")
      color="#ff9638";

    else if(o.type==="magnet")
      color="#9f79f4";

    else
      color="#4dd9e9";

    ctx.shadowBlur=18;
    ctx.shadowColor=color;

    ctx.fillStyle=color;

    ctx.beginPath();

    ctx.arc(
      0,0,r,
      0,
      Math.PI*2
    );

    ctx.fill();

    ctx.shadowBlur=0;

    ctx.fillStyle="#fff";

    ctx.font=
      `bold ${Math.max(13,r*.82)}px Arial`;

    ctx.textAlign="center";

    ctx.fillText(
      o.type==="boost"?"⚡":
      o.type==="magnet"?"🧲":"◇",
      0,
      r*.34
    );
  }


  /* 宝箱 */

  else if(o.type==="chest"){

    let w=31+43*s;
    let h=22+28*s;

    ctx.translate(
      0,
      -h
    );

    ctx.shadowBlur=13;
    ctx.shadowColor="rgba(255,190,50,.65)";

    ctx.fillStyle="#704529";

    ctx.fillRect(
      -w/2,
      0,
      w,
      h
    );

    ctx.fillStyle="#d9a936";

    ctx.fillRect(
      -w*.07,
      0,
      w*.14,
      h
    );

    ctx.fillStyle="#a96b36";

    ctx.beginPath();

    ctx.arc(
      0,
      1,
      w*.47,
      Math.PI,
      Math.PI*2
    );

    ctx.fill();

    ctx.shadowBlur=0;

    ctx.fillStyle="#ffe273";

    ctx.fillRect(
      -5,
      -3,
      10,
      7
    );
  }


  /* 障碍 */

  else{

    let w=25+54*s;
    let h=25+53*s;

    ctx.translate(
      0,
      -h*.42
    );


    /* 岩石 */

    if(o.type==="rock"){

      ctx.fillStyle="#65736e";

      ctx.beginPath();

      ctx.moveTo(
        -w*.56,0
      );

      ctx.lineTo(
        -w*.36,
        -h*.72
      );

      ctx.lineTo(
        0,
        -h
      );

      ctx.lineTo(
        w*.55,
        -h*.40
      );

      ctx.lineTo(
        w*.44,
        0
      );

      ctx.closePath();

      ctx.fill();

      ctx.fillStyle="#8c9990";

      ctx.beginPath();

      ctx.moveTo(
        -w*.36,
        -h*.72
      );

      ctx.lineTo(
        0,
        -h
      );

      ctx.lineTo(
        -w*.02,
        -h*.36
      );

      ctx.lineTo(
        -w*.25,
        -h*.20
      );

      ctx.closePath();

      ctx.fill();
    }


    /* 木头 */

    else if(o.type==="log"){

      ctx.fillStyle="#80502e";

      ctx.fillRect(
        -w*.58,
        -h*.28,
        w*1.16,
        h*.58
      );

      ctx.fillStyle="#b7783e";

      ctx.beginPath();

      ctx.arc(
        -w*.58,
        0,
        h*.25,
        0,
        Math.PI*2
      );

      ctx.fill();

      ctx.strokeStyle="#714326";
      ctx.lineWidth=2;

      ctx.beginPath();

      ctx.arc(
        -w*.58,
        0,
        h*.13,
        0,
        Math.PI*2
      );

      ctx.stroke();
    }


    /* 路障 */

    else if(o.type==="barrier"){

      ctx.fillStyle="#c96742";

      ctx.fillRect(
        -w*.58,
        -h*.55,
        w*1.16,
        h*.65
      );

      ctx.fillStyle="#ffc15c";

      ctx.fillRect(
        -w*.45,
        -h*.38,
        w*.9,
        h*.12
      );

      ctx.fillStyle="#744d37";

      ctx.fillRect(
        -w*.47,
        h*.08,
        w*.10,
        h*.35
      );

      ctx.fillRect(
        w*.37,
        h*.08,
        w*.10,
        h*.35
      );
    }

  }

  ctx.restore();
}


/* =====================================================
   臭臭鼠
===================================================== */

function drawMouse(){

  let x=laneX(G.lane,.94);

  let ground=
    roadY(.94)-8;

  let y=
    ground-G.jump;

  let run=
    Math.sin(G.time*.018);

  let sx=
    1+
    Math.abs(run)*.025;

  let sy=
    1-
    Math.abs(run)*.018;

  ctx.save();

  ctx.translate(x,y);

  ctx.scale(sx,sy);


  /* 阴影 */

  let shadow=
    clamp(
      1-G.jump/130,
      .18,
      1
    );

  ctx.fillStyle=
    `rgba(20,45,35,${.24*shadow})`;

  ctx.beginPath();

  ctx.ellipse(
    0,
    11,
    34*shadow,
    9*shadow,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 尾巴 */

  ctx.strokeStyle="#9c644d";
  ctx.lineWidth=6;
  ctx.lineCap="round";

  ctx.beginPath();

  ctx.moveTo(
    -24,-15
  );

  ctx.quadraticCurveTo(
    -55,
    -4,
    -47,
    17+run*4
  );

  ctx.stroke();


  /* 身体 */

  ctx.fillStyle="#a7745d";

  ctx.beginPath();

  ctx.ellipse(
    0,
    -20,
    27,
    34,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 肚子 */

  ctx.fillStyle="#d7a079";

  ctx.beginPath();

  ctx.ellipse(
    4,
    -15,
    15,
    23,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 腿 */

  ctx.strokeStyle="#815440";
  ctx.lineWidth=6;

  ctx.beginPath();

  ctx.moveTo(
    -12,4
  );

  ctx.lineTo(
    -16+run*6,
    16
  );

  ctx.moveTo(
    12,4
  );

  ctx.lineTo(
    16-run*6,
    16
  );

  ctx.stroke();


  /* 耳朵 */

  ctx.fillStyle="#b87961";

  ctx.beginPath();

  ctx.arc(
    -17,-52,
    13,
    0,
    Math.PI*2
  );

  ctx.arc(
    17,-52,
    13,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.fillStyle="#e9aaa0";

  ctx.beginPath();

  ctx.arc(
    -17,-52,
    7,
    0,
    Math.PI*2
  );

  ctx.arc(
    17,-52,
    7,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 头 */

  ctx.fillStyle="#b77d61";

  ctx.beginPath();

  ctx.ellipse(
    0,
    -52,
    27,
    24,
    0,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 眼睛 */

  ctx.fillStyle="#1c2021";

  ctx.beginPath();

  ctx.arc(
    -9,-56,
    3.6,
    0,
    Math.PI*2
  );

  ctx.arc(
    9,-56,
    3.6,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 鼻子 */

  ctx.fillStyle="#4c3030";

  ctx.beginPath();

  ctx.arc(
    0,-46,
    4,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 嘴 */

  ctx.strokeStyle="#5b3530";
  ctx.lineWidth=2;

  ctx.beginPath();

  ctx.arc(
    0,
    -43,
    7,
    .15,
    Math.PI-.15
  );

  ctx.stroke();


  /* 手 */

  ctx.strokeStyle="#8d5c49";
  ctx.lineWidth=5;

  ctx.beginPath();

  ctx.moveTo(
    -21,-27
  );

  ctx.lineTo(
    -29+run*3,
    -17
  );

  ctx.moveTo(
    21,-27
  );

  ctx.lineTo(
    29-run*3,
    -17
  );

  ctx.stroke();


  /* 护盾 */

  if(G.shield>0){

    ctx.strokeStyle=
      "rgba(75,222,255,.85)";

    ctx.lineWidth=4;

    ctx.fillStyle=
      "rgba(75,222,255,.12)";

    ctx.beginPath();

    ctx.arc(
      0,
      -30,
      53+
      Math.sin(G.time*.01)*4,
      0,
      Math.PI*2
    );

    ctx.fill();

    ctx.stroke();
  }

  ctx.restore();
}


/* =====================================================
   开场专用臭臭鼠
   不再调用正常跑步位置
===================================================== */

function drawIntroMouse(t){

  let progress=
    clamp(t/1.0,0,1);

  let x;

  if(t<1){

    /* 从画面左侧走出来 */

    x=
      lerp(
        -90,
        W/2,
        progress
      );

  }else{

    x=W/2;
  }

  let ground=
    roadY(.94)-8;

  let y=ground;

  let wave=0;

  if(t>=1&&t<2.35){

    wave=
      Math.sin(
        (t-1)*8
      );
  }

  let turn=0;

  if(t>=2.35){

    turn=
      clamp(
        (t-2.35)/.75,
        0,
        1
      );
  }

  ctx.save();

  ctx.translate(x,y);

  /* 先画身体 */

  let run=
    Math.sin(G.time*.015);

  let scale=
    1.05;

  ctx.scale(
    scale,
    scale
  );

  /*
    开场挥手：
    正常鼠鼠基础身体
  */

  ctx.save();

  /* 身体 */

  ctx.fillStyle="#a7745d";

  ctx.beginPath();

  ctx.ellipse(
    0,-20,
    27,34,
    0,0,Math.PI*2
  );

  ctx.fill();

  /* 肚子 */

  ctx.fillStyle="#d7a079";

  ctx.beginPath();

  ctx.ellipse(
    4,-15,
    15,23,
    0,0,Math.PI*2
  );

  ctx.fill();


  /* 耳朵 */

  ctx.fillStyle="#b87961";

  ctx.beginPath();

  ctx.arc(
    -17,-52,
    13,
    0,
    Math.PI*2
  );

  ctx.arc(
    17,-52,
    13,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 头 */

  ctx.fillStyle="#b77d61";

  ctx.beginPath();

  ctx.ellipse(
    0,-52,
    27,24,
    0,0,Math.PI*2
  );

  ctx.fill();


  /* 内耳 */

  ctx.fillStyle="#e9aaa0";

  ctx.beginPath();

  ctx.arc(
    -17,-52,
    7,
    0,
    Math.PI*2
  );

  ctx.arc(
    17,-52,
    7,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 眼睛 */

  ctx.fillStyle="#1b2021";

  ctx.beginPath();

  ctx.arc(
    -9,-56,
    3.5,
    0,
    Math.PI*2
  );

  ctx.arc(
    9,-56,
    3.5,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 鼻子 */

  ctx.fillStyle="#4c3030";

  ctx.beginPath();

  ctx.arc(
    0,-46,
    4,
    0,
    Math.PI*2
  );

  ctx.fill();


  /* 左手 */

  ctx.strokeStyle="#8d5c49";
  ctx.lineWidth=5;
  ctx.lineCap="round";

  ctx.beginPath();

  ctx.moveTo(
    -21,-27
  );

  ctx.lineTo(
    -29,
    -18
  );

  ctx.stroke();


  /* 右手挥手 */

  ctx.save();

  ctx.translate(
    21,
    -27
  );

  ctx.rotate(
    -0.35+
    wave*.5
  );

  ctx.beginPath();

  ctx.moveTo(0,0);

  ctx.lineTo(
    9,
    -25
  );

  ctx.stroke();

  /* 手掌 */

  ctx.fillStyle="#a7745d";

  ctx.beginPath();

  ctx.arc(
    10,
    -28,
    7,
    0,
    Math.PI*2
  );

  ctx.fill();

  ctx.restore();


  /* 嘴 */

  ctx.strokeStyle="#5b3530";
  ctx.lineWidth=2;

  ctx.beginPath();

  if(t>=1&&t<2.35){

    /* 说话状态 */

    ctx.ellipse(
      0,
      -43,
      5,
      4+
      Math.abs(
        Math.sin(t*15)
      )*3,
      0,
      0,
      Math.PI*2
    );

  }else{

    ctx.arc(
      0,
      -43,
      7,
      .15,
      Math.PI-.15
    );
  }

  ctx.stroke();

  ctx.restore();

  /*
    转身效果：
    后半段缩窄，模拟转身
  */

  if(turn>0){

    ctx.fillStyle=
      "rgba(100,65,50,.16)";

    ctx.beginPath();

    ctx.ellipse(
      0,
      -28,
      28*(1-turn*.7),
      40,
      0,
      0,
      Math.PI*2
    );

    ctx.fill();
  }

  ctx.restore();
}


/* =====================================================
   粒子
===================================================== */

function particle(x,y,color){

  G.particles.push({

    x,
    y,

    vx:rand(-2.4,2.4),

    vy:rand(-4,-1),

    life:1,

    color
  });
}


function updateParticles(dt){

  for(
    let i=G.particles.length-1;
    i>=0;
    i--
  ){

    let p=G.particles[i];

    p.x+=p.vx;
    p.y+=p.vy;

    p.vy+=.12;

    p.life-=dt*2;

    if(p.life<=0){

      G.particles.splice(i,1);
    }
  }
}


function drawParticles(){

  for(const p of G.particles){

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


/* =====================================================
   收集
===================================================== */

function collect(o){

  if(o.hit)return;

  o.hit=true;


  if(o.type==="coin"){

    G.coins++;

    for(let i=0;i<5;i++){

      particle(
        laneX(o.lane,o.z),
        roadY(o.z)-30,
        "#ffe15b"
      );
    }
  }


  else if(o.type==="boost"){

    G.boost=5;

    showToast("⚡ 加速！");
  }


  else if(o.type==="magnet"){

    G.magnet=7;

    showToast("🧲 磁铁！");
  }


  else if(o.type==="shield"){

    G.shield=8;

    showToast("🛡 护盾！");
  }


  else if(o.type==="chest"){

    G.coins+=5;

    showToast("🎁 +5金币");

    for(let i=0;i<15;i++){

      particle(
        laneX(o.lane,o.z),
        roadY(o.z)-35,
        "#ffd84a"
      );
    }
  }
}


/* =====================================================
   碰撞
===================================================== */

function checkObjects(){

  for(const o of G.objects){

    if(o.hit)continue;

    /*
      只有进入玩家附近区域才判断。
    */

    if(o.z<.88)continue;

    if(o.z>1.01)continue;


    /* 金币 */

    if(o.type==="coin"){

      if(G.magnet>0){

        if(
          Math.abs(
            G.lane-o.lane
          )<=1
        ){

          collect(o);
        }

      }else if(
        G.lane===o.lane
      ){

        collect(o);
      }

      continue;
    }


    /* 道具 */

    if(
      o.type==="boost"||
      o.type==="magnet"||
      o.type==="shield"||
      o.type==="chest"
    ){

      if(
        G.lane===o.lane
      ){

        collect(o);
      }

      continue;
    }


    /* 障碍 */

    if(
      G.lane===o.lane
    ){

      /*
        跳起来可以躲开
      */

      if(G.jump>35){

        continue;
      }


      if(G.shield>0){

        G.shield=0;

        o.hit=true;

        for(let i=0;i<14;i++){

          particle(
            laneX(o.lane,o.z),
            roadY(o.z)-25,
            "#6de8ff"
          );
        }

        showToast("🛡 挡住了！");

      }else{

        gameOver();

        return;
      }
    }
  }
}


/* =====================================================
   开场
===================================================== */

function introUpdate(dt){

  G.introTime+=dt;

  $("skip").style.display="block";

  let t=G.introTime;


  if(t<1){

    $("introText").style.opacity=0;
  }

  else if(t<2.35){

    $("introText").style.opacity=1;
  }

  else if(t<3.1){

    $("introText").style.opacity=0;
  }


  /*
    4秒后正式进入跑酷
  */

  if(t>=4){

    $("skip").style.display="none";

    $("introText").style.opacity=0;

    G.mode="run";
  }
}


$("skip").onclick=()=>{

  G.mode="run";

  $("skip").style.display="none";

  $("introText").style.opacity=0;
};


/* =====================================================
   开始
===================================================== */

function startGame(){

  G.mode="intro";

  G.time=0;

  G.last=performance.now();

  G.distance=0;

  G.coins=0;

  G.lane=1;

  G.targetLane=1;

  G.jump=0;

  G.jumpVelocity=0;

  /*
    初始速度故意慢
  */

  G.speed=.055;

  G.objects=[];

  G.particles=[];

  G.spawnTimer=2.2;

  G.shield=0;
  G.magnet=0;
  G.boost=0;

  G.shake=0;

  G.introTime=0;

  G.worldOffset=0;

  $("startScreen")
    .classList
    .add("hidden");

  $("gameOver")
    .classList
    .add("hidden");

  $("hint").style.opacity=0;
}


$("startBtn").onclick=startGame;

$("restartBtn").onclick=startGame;


/* =====================================================
   死亡
===================================================== */

function gameOver(){

  if(
    G.mode==="gameover"
  )return;

  G.mode="gameover";

  G.shake=.45;

  let d=
    Math.floor(G.distance);

  if(d>G.best){

    G.best=d;

    localStorage.setItem(
      "chouchou_best",
      G.best
    );
  }

  $("finalDistance")
    .textContent=d+"m";

  $("finalCoins")
    .textContent=G.coins;

  $("bestDistance")
    .textContent=G.best;

  $("gameOver")
    .classList
    .remove("hidden");

  for(let i=0;i<25;i++){

    particle(
      W/2,
      H*.68,
      "#ffffff"
    );
  }
}


/* =====================================================
   HUD
===================================================== */

function updateHUD(){

  $("distance")
    .textContent=
    Math.floor(G.distance);

  $("coins")
    .textContent=
    G.coins;

  $("bestDistance")
    .textContent=
    G.best;

  let status="正常";

  if(G.boost>0)
    status="⚡ 加速";

  else if(G.magnet>0)
    status="🧲 磁铁";

  else if(G.shield>0)
    status="🛡 护盾";

  $("status")
    .textContent=status;

  let power=
    Math.max(
      G.boost,
      G.magnet,
      G.shield
    );

  $("powerFill")
    .style
    .width=
    Math.min(
      power/8*100,
      100
    )+"%";
}


/* =====================================================
   游戏更新
===================================================== */

function update(dt){

  G.time+=dt*1000;

  /*
    开场
  */

  if(G.mode==="intro"){

    introUpdate(dt);

    return;
  }


  if(G.mode!=="run"){

    return;
  }


  /*
    难度曲线

    0-100：很慢
    100-300：慢慢增加
    300-600：正常
    600-1000：偏快
    1000+：挑战
  */

  let d=G.distance;

  let baseSpeed;

  if(d<100){

    baseSpeed=
      lerp(
        .055,
        .065,
        d/100
      );

  }else if(d<300){

    baseSpeed=
      lerp(
        .065,
        .078,
        (d-100)/200
      );

  }else if(d<600){

    baseSpeed=
      lerp(
        .078,
        .092,
        (d-300)/300
      );

  }else if(d<1000){

    baseSpeed=
      lerp(
        .092,
        .108,
        (d-600)/400
      );

  }else{

    baseSpeed=
      Math.min(
        .12,
        .108+
        (d-1000)*.000015
      );
  }

  G.speed=baseSpeed;


  /*
    加速
  */

  if(G.boost>0){

    G.speed+=.035;

    G.boost-=dt;

    if(G.boost<0)
      G.boost=0;
  }


  /*
    距离
  */

  G.distance+=
    G.speed*
    dt*
    60;


  G.worldOffset+=
    G.speed*
    dt*
    60;


  /*
    换道
  */

  G.lane=
    lerp(
      G.lane,
      G.targetLane,
      clamp(dt*10,0,1)
    );


  /*
    跳跃
  */

  if(
    G.jump>0||
    G.jumpVelocity>0
  ){

    G.jump+=
      G.jumpVelocity*
      dt*
      60;

    G.jumpVelocity-=
      .62*
      dt*
      60;

    if(G.jump<=0){

      G.jump=0;

      G.jumpVelocity=0;

      for(let i=0;i<6;i++){

        particle(
          laneX(G.lane,.94),
          roadY(.94)-5,
          "#d6c099"
        );
      }
    }
  }


  /*
    道具时间
  */

  if(G.magnet>0){

    G.magnet-=dt;

    if(G.magnet<0)
      G.magnet=0;
  }

  if(G.shield>0){

    G.shield-=dt;

    if(G.shield<0)
      G.shield=0;
  }


  /*
    生成
  */

  updateSpawn(dt);


  /*
    世界向玩家移动
  */

  for(const o of G.objects){

    /*
      这里故意不要太快。
    */

    o.z-=
      G.speed*
      dt*
      .63;
  }


  /*
    磁铁
  */

  if(G.magnet>0){

    for(const o of G.objects){

      if(
        o.type==="coin"&&
        !o.hit&&
        o.z>.60&&
        Math.abs(
          o.lane-G.lane
        )<=1
      ){

        collect(o);
      }
    }
  }


  checkObjects();


  /*
    清理
  */

  G.objects=
    G.objects.filter(
      o=>
      o.z>-.12&&!o.hit
    );


  updateParticles(dt);


  if(G.shake>0){

    G.shake-=dt;
  }


  updateHUD();


  /*
    100m提示一次
  */

  if(
    G.distance>8&&
    G.distance<10
  ){

    $("hint").style.opacity=.8;

    setTimeout(()=>{
      $("hint").style.opacity=0;
    },1800);
  }
}


/* =====================================================
   操作
===================================================== */

let touchX=0;
let touchY=0;

canvas.addEventListener(
  "touchstart",
  e=>{

    if(!e.touches.length)return;

    touchX=
      e.touches[0].clientX;

    touchY=
      e.touches[0].clientY;

  },
  {passive:true}
);


canvas.addEventListener(
  "touchend",
  e=>{

    if(G.mode!=="run")return;

    if(!e.changedTouches.length)
      return;

    let x=
      e.changedTouches[0].clientX;

    let y=
      e.changedTouches[0].clientY;

    let dx=x-touchX;
    let dy=y-touchY;


    if(
      Math.abs(dx)<30&&
      Math.abs(dy)<30
    ){

      return;
    }


    /*
      左右
    */

    if(
      Math.abs(dx)>
      Math.abs(dy)
    ){

      if(dx>0){

        G.targetLane=
          clamp(
            G.targetLane+1,
            0,2
          );

      }else{

        G.targetLane=
          clamp(
            G.targetLane-1,
            0,2
          );
      }

    }


    /*
      上下
    */

    else{

      if(dy<0){

        if(G.jump===0){

          G.jump=1;

          G.jumpVelocity=2.65;
        }

      }else{

        G.jumpVelocity-=1.5;
      }
    }

  },
  {passive:true}
);


/* 键盘 */

addEventListener(
  "keydown",
  e=>{

    if(e.key==="ArrowLeft"){

      G.targetLane=
        clamp(
          G.targetLane-1,
          0,2
        );
    }


    if(e.key==="ArrowRight"){

      G.targetLane=
        clamp(
          G.targetLane+1,
          0,2
        );
    }


    if(
      e.key==="ArrowUp"||
      e.key===" "
    ){

      if(G.jump===0){

        G.jump=1;

        G.jumpVelocity=2.65;
      }
    }


    if(e.key==="ArrowDown"){

      G.jumpVelocity-=1.5;
    }
  }
);


/* =====================================================
   Toast
===================================================== */

let toastTimer=0;

function showToast(text){

  $("toast")
    .textContent=text;

  $("toast")
    .style
    .opacity=1;

  toastTimer=1;
}


/* =====================================================
   绘制
===================================================== */

function draw(){

  ctx.save();


  /*
    死亡震动
  */

  if(G.shake>0){

    ctx.translate(
      rand(-5,5),
      rand(-5,5)
    );
  }


  drawSky();

  drawSea();

  drawIsland();

  drawRoad();

  drawEnvironment();


  /*
    远处 → 近处
  */

  let arr=
    [...G.objects]
    .sort(
      (a,b)=>
      a.z-b.z
    );


  for(const o of arr){

    drawObject(o);
  }


  /*
    正常游戏角色
  */

  if(G.mode!=="intro"){

    drawMouse();
  }


  drawParticles();

  ctx.restore();


  /*
    开场动画独立绘制
  */

  if(G.mode==="intro"){

    let t=G.introTime;

    drawIntroMouse(t);

    /*
      对话文字
    */

    if(t>=1&&t<2.35){

      $("introText")
        .style
        .opacity=1;

    }else{

      $("introText")
        .style
        .opacity=0;
    }
  }
}


/* =====================================================
   主循环
===================================================== */

function loop(now){

  let dt=
    Math.min(
      (now-G.last)/1000,
      .033
    );

  G.last=now;

  update(dt);

  draw();

  requestAnimationFrame(loop);
}


G.last=
  performance.now();

requestAnimationFrame(loop);

</script>

</body>
</html>
