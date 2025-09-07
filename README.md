<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>跳一跳俯视角·赚钱版</title>
<meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=no">
<style>
*{box-sizing:border-box;margin:0;padding:0}
body{display:flex;justify-content:center;align-items:center;height:100vh;background:#1e1e1e;font-family:Helvetica;overflow:hidden}
#c{background:#fff;border-radius:12px;box-shadow:0 0 20px rgba(0,0,0,.5)}
#ui{position:absolute;top:10px;left:50%;transform:translateX(-50%);color:#fff;text-align:center}
#score{font-size:32px}#best{font-size:14px;opacity:.8}
#power{position:absolute;bottom:20px;left:50%;transform:translateX(-50%);width:200px;height:10px;background:rgba(255,255,255,.3);border-radius:5px;overflow:hidden}
#powerBar{height:100%;background:linear-gradient(90deg,#f093fb 0%,#f5576c 100%);width:0%}
</style>
</head>
<body>
<div id="ui"><div id="score">0</div><div id="best">最佳 0</div></div>
<canvas id="c"></canvas>
<div id="power"><div id="powerBar"></div></div>
<script>
/*************** 俯视角跳一跳 ****************/
const ctx = c.getContext('2d'), DPR = window.devicePixelRatio||1;
function resize(){
  const rect = c.getBoundingClientRect();
  c.width = rect.width * DPR;
  c.height = rect.height * DPR;
  ctx.scale(DPR,DPR);
  c.style.width = rect.width + 'px';
  c.style.height = rect.height + 'px';
}
resize(); window.addEventListener('resize',resize);

let score = 0, best = +(localStorage.best||0), pressing=false, power=0, gameOver=false;
const g = 0.6, jumpMax = 18;
const player = {x:0,y:0,z:50,r:18,color:'#ff0055'};
let platforms = [{x:0,y:0,w:120,h:120,z:0,color:'#444'}];
let cam = {x:0,y:0}, shake=0;

// 工具
const R = (l,u)=>Math.random()*(u-l)+l;
const particle = (x,y,n=15)=>{
  for(let i=0;i<n;i++){
    const a = R(0,6.28), s = R(2,6), sz = R(3,8);
    ctx.save();
    ctx.globalAlpha = .9;
    ctx.fillStyle = '#ffd93d';
    ctx.translate(x,y);
    ctx.rotate(a);
    ctx.fillRect(0,0,sz,sz);
    ctx.restore();
  }
};

// 生成新平台
const addPlatform = ()=>{
  const last = platforms[platforms.length-1];
  const dir = R(0,1)>0.5? {x:1,y:0}:{x:0,y:1}; // 只向右或向上
  const gap = R(80,140);
  platforms.push({
    x: last.x + dir.x*gap,
    y: last.y + dir.y*gap,
    w:110, h:110, z:20,
    color: `hsl(${R(0,360)},70%,50%)`
  });
};
for(let i=0;i<5;i++) addPlatform();

// 画平台（俯视角正方形）
const drawPlatform = p=>{
  ctx.save();
  ctx.translate(p.x-cam.x+shake*(Math.random()-.5)*10, p.y-cam.y+shake*(Math.random()-.5)*10);
  // 顶面
  ctx.fillStyle = p.color;
  ctx.fillRect(-p.w/2,-p.h/2,p.w,p.h);
  // 阴影
  ctx.fillStyle = 'rgba(0,0,0,.15)';
  ctx.fillRect(-p.w/2+5,-p.h/2+5,p.w,p.h);
  ctx.restore();
};

// 画玩家（圆+朝向箭头）
const drawPlayer = ()=>{
  ctx.save();
  ctx.translate(player.x-cam.x+shake*(Math.random()-.5)*10, player.y-cam.y+shake*(Math.random()-.5)*10);
  // 阴影
  ctx.globalAlpha = .3;
  ctx.fillStyle = '#000';
  ctx.beginPath(); ctx.arc(0,8,player.r,0,6.28); ctx.fill();
  // 身体
  ctx.globalAlpha = 1;
  ctx.fillStyle = player.color;
  ctx.beginPath(); ctx.arc(0,0,player.r,0,6.28); ctx.fill();
  // 箭头
  ctx.strokeStyle='#fff'; ctx.lineWidth=3;
  ctx.beginPath(); ctx.moveTo(0,-8); ctx.lineTo(0,8); ctx.moveTo(-5,3); ctx.lineTo(0,8); ctx.lineTo(5,3); ctx.stroke();
  ctx.restore();
};

// 主循环
function loop(){
  ctx.clearRect(0,0,c.width,c.height);
  // 画平台
  platforms.forEach(drawPlatform);
  // 画玩家
  drawPlayer();
  if(!gameOver){
    // 物理
    player.z -= g;
    if(player.z<50){ player.z=50; onPlatform=true; }else onPlatform=false;
    // 镜头跟随
    cam.x += (player.x-cam.x)*0.12;
    cam.y += (player.y-cam.y)*0.12;
    // 蓄力条
    if(pressing){ power=Math.min(power+1.2,60); powerBar.style.width=power+'%'; }
    // 掉落检测
    if(player.y>400){ gameOver=true; showAd(); }
    requestAnimationFrame(loop);
  }
}

// 跳跃（四方向）
function jump(dx,dy){
  if(gameOver){location.reload();return;}
  if(!onPlatform) return;
  const spd = 4 + power/12;
  player.x += dx*spd*16;
  player.y += dy*spd*16;
  particle(player.x-cam.x, player.y-cam.y);
  shake=8; power=0; powerBar.style.width='0%';
  score++; scoreEl.innerText=score; if(score>best){best=score;localStorage.best=best;bestEl.innerText='最佳 '+best;}
}
window.onkeydown = e=>{
  if(pressing) return;
  switch(e.key){
    case 'w':case 'ArrowUp':    jump(0,-1); break;
    case 's':case 'ArrowDown':  jump(0,1);  break;
    case 'a':case 'ArrowLeft':  jump(-1,0); break;
    case 'd':case 'ArrowRight': jump(1,0);  break;
    case ' ': pressing=true; e.preventDefault(); break;
  }
};
window.onkeyup = e=>{
  if(e.key===' ') pressing=false;
};
// 手机点击四方向
let baseW = c.getBoundingClientRect().width/2, baseH = c.getBoundingClientRect().height/2;
c.addEventListener('touchstart',e=>{
  const t = e.touches[0], rect = c.getBoundingClientRect();
  const dx = (t.clientX - rect.left - baseW)/baseW;
  const dy = (t.clientY - rect.top - baseH)/baseH;
  if(Math.abs(dx)>Math.abs(dy)) jump(dx>0?1:-1,0);
  else jump(0,dy>0?1:-1);
});

// 屏幕震动递减
setInterval(()=>shake*=0.9,50);

// 广告复活
function showAd(){
  // 替换成你的真实广告ID
  if(confirm('看广告复活？点确定继续游戏')){
    // 这里放谷歌或微信激励视频代码，先简单reload
    location.reload();
  }
}
const scoreEl=document.getElementById('score'), bestEl=document.getElementById('best');
bestEl.innerText='最佳 '+best;
loop();
</script>
</body>
</html>
