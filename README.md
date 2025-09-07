<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>3D跳一跳·赚钱版</title>
<style>
body{margin:0;background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);display:flex;justify-content:center;align-items:center;height:100vh;overflow:hidden;font-family:Helvetica}
#c{background:#fff;border-radius:12px;box-shadow:0 12px 40px rgba(0,0,0,.3)}
#score{position:absolute;top:20px;left:50%;transform:translateX(-50%);font-size:42px;color:#fff;text-shadow:0 2px 4px rgba(0,0,0,.2)}
#power{position:absolute;bottom:30px;left:50%;transform:translateX(-50%);width:220px;height:16px;background:rgba(255,255,255,.3);border-radius:8px;overflow:hidden}
#powerBar{height:100%;background:linear-gradient(90deg,#f093fb 0%,#f5576c 100%);width:0%;transition:width .05s}
</style>
</head>
<body>
<div id="score">0</div>
<canvas id="c" width="480" height="720"></canvas>
<div id="power"><div id="powerBar"></div></div>
<script>
/*************** 3D 跳一跳核心代码 ****************/
const ctx = c.getContext('2d');
let score = 0, best = localStorage.best||0, pressing=false, power=0, gameOver=false;
const g = 0.6, F = Math.PI/180; // 重力 & 角度常数
const player = {x:120,y:500,z:50,r:18,vx:0,vy:0,color:'#ff0055',trail:[]};
let platforms = [{x:100,y:600,w:120,h:25,z:0,color:'#444'}];
let cam = {x:0,y:0}; // 镜头偏移

// 工具函数
const R = (l,u)=>Math.random()*(u-l)+l;
const drawBox = (p)=>{
  ctx.save();
  ctx.translate(p.x-cam.x, p.y-cam.y);
  // 3D 顶面
  ctx.fillStyle = p.color;
  ctx.beginPath();
  ctx.moveTo(0, -p.z);
  ctx.lineTo(p.w, -p.z);
  ctx.lineTo(p.w + p.z/2, -p.z + p.z/2);
  ctx.lineTo(p.z/2, -p.z + p.z/2);
  ctx.closePath(); ctx.fill();
  // 3D 右侧面
  ctx.fillStyle = shade(p.color,.8);
  ctx.beginPath();
  ctx.moveTo(p.w, 0);
  ctx.lineTo(p.w, -p.z);
  ctx.lineTo(p.w + p.z/2, -p.z + p.z/2);
  ctx.lineTo(p.w + p.z/2, p.z/2);
  ctx.closePath(); ctx.fill();
  // 正面
  ctx.fillStyle = shade(p.color,.6);
  ctx.fillRect(0,0,p.w,p.h);
  ctx.restore();
};
const shade = (col,f)=>{
  const r = parseInt(col.slice(1,3),16)*f|0;
  const g = parseInt(col.slice(3,5),16)*f|0;
  const b = parseInt(col.slice(5,7),16)*f|0;
  return `rgb(${r},${g},${b})`;
};
const particle = (x,y)=>{
  for(let i=0;i<20;i++){
    const a = R(0,6.28), s = R(1,4);
    ctx.save();
    ctx.globalAlpha = .8;
    ctx.fillStyle = '#ffd93d';
    ctx.beginPath();
    ctx.arc(x+Math.cos(a)*s, y+Math.sin(a)*s, 2, 0, 6.28);
    ctx.fill();
    ctx.restore();
  }
};

// 生成新平台
const addPlatform = ()=>{
  const last = platforms[platforms.length-1];
  const gap = R(90,160), w = R(80,140);
  platforms.push({
    x: last.x + gap,
    y: last.y + R(-80,80),
    w, h:25, z:20,
    color: `hsl(${R(0,360)},70%,50%)`
  });
};
for(let i=0;i<6;i++) addPlatform();

// 主循环
function loop(){
  ctx.clearRect(0,0,c.width,c.height);
  // 画平台
  platforms.forEach(drawBox);
  // 画玩家阴影
  ctx.save();
  ctx.globalAlpha = .2;
  ctx.fillStyle = '#000';
  ctx.beginPath();
  ctx.ellipse(player.x-cam.x, player.y-cam.y+5, player.r, player.r*0.4, 0, 0, 6.28);
  ctx.fill();
  ctx.restore();
  // 画玩家
  ctx.save();
  ctx.translate(player.x-cam.x, player.y-cam.y);
  ctx.fillStyle = player.color;
  ctx.beginPath();
  ctx.arc(0,0,player.r,0,6.28);
  ctx.fill();
  // 笑脸
  ctx.fillStyle='#fff';
  ctx.beginPath(); ctx.arc(-6,-4,3,0,6.28); ctx.fill();
  ctx.beginPath(); ctx.arc(6,-4,3,0,6.28); ctx.fill();
  ctx.strokeStyle='#fff'; ctx.lineWidth=2; ctx.beginPath(); ctx.arc(0,3,8,0.2,2.9); ctx.stroke();
  ctx.restore();

  if(!gameOver){
    // 物理
    if(pressing){ power = Math.min(power+1.2,60); powerBar.style.width = power+'%'; }
    player.vy += g;
    player.x += player.vx; player.y += player.vy;
    // 碰撞检测
    let onPlatform = false;
    platforms.forEach(p=>{
      if(player.x+player.r>p.x && player.x-player.r<p.x+p.w &&
         player.y+player.r>p.y && player.y-player.r<p.y+p.h){
        if(player.vy>0){
          player.y = p.y - player.r;
          player.vy = 0;
          onPlatform = true;
        }
      }
    });
    if(!onPlatform && player.y>600){ gameOver=true; showAd(); }
    // 镜头跟随
    cam.x += (player.x-240-cam.x)*0.08;
    cam.y += (player.y-360-cam.y)*0.08;
    // 计分
    if(player.x>platforms[1].x){ score++; platforms.shift(); addPlatform(); }
    requestAnimationFrame(loop);
  }
}

// 操作
c.onmousedown = c.ontouchstart = e=>{e.preventDefault();if(!gameOver)pressing=true;power=0;};
c.onmouseup = c.ontouchend = e=>{
  e.preventDefault();
  if(gameOver){location.reload();return;}
  pressing=false;
  const rad = power*F*2.2, spd = power*0.28;
  player.vx = Math.cos(rad)*spd;
  player.vy = -Math.sin(rad)*spd;
  particle(player.x-cam.x, player.y-cam.y);
  powerBar.style.width='0%'; power=0;
};
loop();
</script>
</body>
</html>
