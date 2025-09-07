# jump-game
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>跳一跳赚钱版</title>
    <style>
        body{margin:0;background:#87CEEB;display:flex;justify-content:center;align-items:center;height:100vh;overflow:hidden;}
        canvas{background:#fff;box-shadow:0 0 10px rgba(0,0,0,0.2);}
    </style>
</head>
<body>
<canvas id="c" width="400" height="600"></canvas>
<script>
//  ======  游戏核心代码（直接复制）  ======
const ctx = c.getContext('2d');
let player = {x:100,y:400,w:30,h:30,color:'#ff6b6b'};
let platforms = [{x:50,y:500,w:100,h:20,color:'#4ecdc4'}];
let score = 0, gameOver = false, pressing = false, power = 0;

// 画矩形
function drawRect(r){ctx.fillStyle=r.color;ctx.fillRect(r.x,r.y,r.w,r.h);}

// 随机生成平台
function addPlatform(){
    const last = platforms[platforms.length-1];
    platforms.push({
        x: last.x + 100 + Math.random()*150,
        y: 500 - Math.random()*200,
        w: 60 + Math.random()*40,
        h: 20,
        color: '#4ecdc4'
    });
}

// 初始化5个平台
for(let i=0;i<5;i++) addPlatform();

// 主循环
function loop(){
    ctx.clearRect(0,0,c.width,c.height);
    // 画平台
    platforms.forEach(drawRect);
    // 画玩家
    drawRect(player);
    // 画力度条
    if(pressing){
        ctx.fillStyle='rgba(255,0,0,0.5)';
        ctx.fillRect(player.x-20,player.y-30,power*2,10);
    }
    // 失败检测
    if(player.y>600){gameOver=true; showAd();}
    requestAnimationFrame(loop);
}

// 按压控制
c.onmousedown = ()=>{if(!gameOver) pressing=true; power=0;};
c.onmouseup = ()=>{
    if(gameOver){location.reload();return;}
    pressing=false;
    // 跳跃
    player.x += power*3;
    player.y -= power*2;
    score++;
    // 镜头跟随
    if(player.y<200){
        platforms.forEach(p=>p.y+=200-player.y);
        player.y=200;
    }
    // 生成新平台
    if(platforms[platforms.length-1].x<500) addPlatform();
};
// 力度增长
setInterval(()=>{if(pressing&&power<50)power+=2;},50);

//  ======  广告代码（Google AdSense激励视频）  ======
function showAd(){
    // 这里是“假广告”，先跑通流程，后面再替换成真广告ID
    if(confirm('看广告复活？点“确定”继续游戏')){
        location.reload();  // 真广告替换这里
    }
}
loop();
</script>
</body>
</html>
