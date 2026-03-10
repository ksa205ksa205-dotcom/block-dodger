
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>BlockDodger</title>
<style>
body{margin:0;color:white;font-family:Arial;text-align:center;background:linear-gradient(180deg,#05070f,#0b1a3a,#02030a);overflow:hidden}
#logo{font-size:42px;font-weight:bold;margin-top:20px;color:#00ff9c;letter-spacing:3px}
canvas{background:transparent;display:block;margin:20px auto;border:2px solid #555}
button{padding:10px 20px;font-size:16px;margin-top:10px;cursor:pointer}
input{padding:8px;font-size:16px}
#menu,#gameover,#shop{position:absolute;top:50%;left:50%;transform:translate(-50%,-50%)}
#info{font-size:14px;color:#aaa;margin-top:10px}
.shop-item{margin:10px}
</style>
</head>
<body>

<div id="logo">BLOCKDODGER</div>

<div id="menu">
<p>Enter Your Name</p>
<input id="name1" placeholder="Player">
<br><br>
<button onclick="startGame()">Start Game</button>
<br><br>
<button onclick="openShop()">Shop</button>
<p id="info">Move: A / D or ← →</p>
</div>

<div id="shop" style="display:none">
<h2>Shop</h2>
<p>Coins: <span id="coinText">0</span></p>
<div class="shop-item">
Speed Boost (10 coins)
<br>
<button onclick="buySpeed()">Buy</button>
</div>
<div class="shop-item">
Shield Power (15 coins)
<br>
<button onclick="buyShield()">Buy</button>
</div>
<br>
<button onclick="closeShop()">Back</button>
</div>

<div id="gameover" style="display:none">
<h2>Game Over</h2>
<p id="scoreText"></p>
<button onclick="restartGame()">Restart</button>
</div>

<canvas id="game" width="400" height="500"></canvas>

<audio id="music" loop>
<source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_c8a8e9b1f0.mp3?filename=8-bit-arcade-110958.mp3" type="audio/mpeg">
</audio>

<script>
const canvas=document.getElementById('game');
const ctx=canvas.getContext('2d');
const music=document.getElementById('music');

let player=null;
let blocks=[];
let powerups=[];
let stars=[];
let keys={};
let frame=0;
let score=0;

/* LOAD SAVED DATA */
let coins=parseInt(localStorage.getItem("bd_coins"))||0;
let speedLevel=parseInt(localStorage.getItem("bd_speed"))||0;
let shieldOwned=(localStorage.getItem("bd_shield")==="true");

let gameRunning=false;
let shieldActive=false;

function saveGame(){
 localStorage.setItem("bd_coins",coins);
 localStorage.setItem("bd_speed",speedLevel);
 localStorage.setItem("bd_shield",shieldOwned);
}

for(let i=0;i<80;i++){
 stars.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,size:Math.random()*2,speed:0.3+Math.random()*0.7});
}

document.addEventListener('keydown',e=>keys[e.key.toLowerCase()]=true);
document.addEventListener('keyup',e=>keys[e.key.toLowerCase()]=false);

function startGame(){
const name=document.getElementById('name1').value||"Player";

player={name:name,x:180,y:460,w:40,h:20,color:"lime",alive:true};

blocks=[];
powerups=[];
frame=0;
score=0;
shieldActive=false;

gameRunning=true;
document.getElementById('menu').style.display='none';
document.getElementById('gameover').style.display='none';

music.currentTime=0;
music.play();
}

function restartGame(){
document.getElementById('menu').style.display='block';
document.getElementById('gameover').style.display='none';
gameRunning=false;
music.pause();
}

function openShop(){
document.getElementById('menu').style.display='none';
document.getElementById('shop').style.display='block';
updateCoins();
}

function closeShop(){
document.getElementById('shop').style.display='none';
document.getElementById('menu').style.display='block';
}

function updateCoins(){
document.getElementById('coinText').textContent=coins;
}

function buySpeed(){
if(coins>=10){
coins-=10;
speedLevel++;
updateCoins();
saveGame();
}
}

function buyShield(){
if(coins>=15){
coins-=15;
shieldOwned=true;
updateCoins();
saveGame();
}
}

function spawnBlock(){
blocks.push({x:Math.random()*360,y:-20,w:40,h:20,speed:2+Math.random()*3});
}

function spawnPower(){
 const types=["coin","shield","slow"];
 const t=types[Math.floor(Math.random()*types.length)];
 powerups.push({x:Math.random()*360,y:-20,w:20,h:20,type:t,speed:2});
}

function update(){

stars.forEach(s=>{
 s.y+=s.speed;
 if(s.y>canvas.height){s.y=0;s.x=Math.random()*canvas.width}
});

if(!gameRunning) return;

frame++;

let moveSpeed=6+(speedLevel*1);

if(keys['a']||keys['arrowleft']) player.x-=moveSpeed;
if(keys['d']||keys['arrowright']) player.x+=moveSpeed;

if(keys[' '] && shieldOwned) shieldActive=true;

player.x=Math.max(0,Math.min(canvas.width-player.w,player.x));

if(frame%40===0) spawnBlock();
if(frame%120===0) spawnPower();

blocks.forEach(b=>b.y+=b.speed);
powerups.forEach(p=>p.y+=p.speed);

blocks.forEach(b=>{
 if(player.alive && player.x<b.x+40 && player.x+40>b.x && player.y<b.y+20 && player.y+20>b.y){

  if(shieldActive){
   shieldActive=false;
   return;
  }

  player.alive=false;
  gameRunning=false;
  music.pause();

  let earned=Math.floor(score/20);
  coins+=earned;
  saveGame();

  document.getElementById('scoreText').textContent="Score: "+score+" | Coins Earned: "+earned;
  document.getElementById('gameover').style.display='block';
 }
});

powerups.forEach(p=>{
 if(player.x<p.x+20 && player.x+40>p.x && player.y<p.y+20 && player.y+20>p.y){

  if(p.type==="coin"){
   coins++;
  }

  if(p.type==="shield"){
   shieldOwned=true;
   shieldActive=true;
  }

  if(p.type==="slow"){
   blocks.forEach(b=>b.speed*=0.5);
   setTimeout(()=>{
     blocks.forEach(b=>b.speed*=2);
   },3000);
  }

  p.collected=true;
 }
});

powerups=powerups.filter(p=>!p.collected && p.y<canvas.height);
blocks=blocks.filter(b=>b.y<canvas.height+40);

score++;
}

function draw(){
ctx.clearRect(0,0,canvas.width,canvas.height);

ctx.fillStyle='white';
stars.forEach(s=>ctx.fillRect(s.x,s.y,s.size,s.size));

if(player && player.alive){
 ctx.save();
 ctx.translate(player.x+20,player.y+10);

 // glow
 ctx.shadowColor='lime';
 ctx.shadowBlur=15;

 // ship body
 ctx.fillStyle=shieldActive?"cyan":"lime";
 ctx.beginPath();
 ctx.moveTo(0,-12);
 ctx.lineTo(14,10);
 ctx.lineTo(-14,10);
 ctx.closePath();
 ctx.fill();

 // cockpit
 ctx.shadowBlur=0;
 ctx.fillStyle='black';
 ctx.beginPath();
 ctx.arc(0,-2,4,0,Math.PI*2);
 ctx.fill();

 ctx.restore();

 ctx.fillStyle='white';
 ctx.font='12px Arial';
 ctx.fillText(player.name,player.x,player.y-8);
}

ctx.fillStyle='red';
blocks.forEach(b=>ctx.fillRect(b.x,b.y,40,20));

powerups.forEach(p=>{
 if(p.type==="coin") ctx.fillStyle='yellow';
 if(p.type==="shield") ctx.fillStyle='cyan';
 if(p.type==="slow") ctx.fillStyle='purple';
 ctx.fillRect(p.x,p.y,20,20);
});

ctx.fillStyle='white';
ctx.font='16px Arial';
ctx.fillText("Score: "+score,10,20);
ctx.fillText("Coins: "+coins,10,40);
}

function loop(){
update();
draw();
requestAnimationFrame(loop);
}

updateCoins();
loop();
</script>

</body>
</html>
