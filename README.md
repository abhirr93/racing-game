
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Racing Game</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#202830;color:white;font-family:Arial;text-align:center}
canvas{display:block;width:100%;height:65vh;background:#74b9e8}
button{padding:14px;margin:5px;border:0;border-radius:12px;font-size:17px;font-weight:bold}
#controls{display:flex;justify-content:space-around;flex-wrap:wrap}
</style>
</head>
<body>
<h2>🏎️ Racing Adventure</h2>
<p>🪙 Coins: <span id="coins">0</span>
 | ⚡ Nitro: <span id="nitro">100</span>%</p>
<canvas id="game"></canvas>
<div id="controls">
<button id="left">⬅️ Left</button>
<button id="boost">⚡ BOOST</button>
<button id="right">Right ➡️</button>
<button id="vehicle">🚗 Change Vehicle</button>
</div>
<p id="status">Collect coins and avoid traffic!</p>
<script>
const canvas=document.getElementById("game");
const ctx=canvas.getContext("2d");
let w,h,carX=0,coins=0,nitro=100;
let roadY=0,objects=[],vehicle=0,boost=false;
const vehicles=["🚗","🏍️","🚙","🚚"];
function resize(){
  w=canvas.width=innerWidth*devicePixelRatio;
  h=canvas.height=innerHeight*.65*devicePixelRatio;
  ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
  w=innerWidth;h=innerHeight*.65;
}
resize();addEventListener("resize",resize);
function spawn(){
  objects.push({x:Math.random()*0.7+0.15,y:-20,
    coin:Math.random()<.65});
}
function draw(){
  ctx.clearRect(0,0,w,h);
  ctx.fillStyle="#388b45";ctx.fillRect(0,0,w,h);
  ctx.fillStyle="#424850";
  ctx.beginPath();
  ctx.moveTo(w*.18,0);ctx.lineTo(w*.82,0);
  ctx.lineTo(w,h);ctx.lineTo(0,h);ctx.fill();
  ctx.strokeStyle="white";ctx.lineWidth=4;
  ctx.setLineDash([30,25]);
  ctx.beginPath();ctx.moveTo(w*.5,roadY%55);
  ctx.lineTo(w*.5,h);ctx.stroke();ctx.setLineDash([]);
  roadY+=boost?12:6;
  objects.forEach(o=>{
    o.y+=boost?5:3;
    ctx.font="30px Arial";
    ctx.textAlign="center";
    ctx.fillText(o.coin?"🪙":"🚙",o.x*w,o.y);
    if(!o.hit&&o.y>h-100&&o.y<h-45&&
       Math.abs(o.x-(.5+carX*.25))<.12){
      o.hit=true;
      if(o.coin){coins+=1;document.getElementById("coins").textContent=coins;}
      else document.getElementById("status").textContent="Traffic! Keep steering!";
    }
  });
  objects=objects.filter(o=>o.y<h+30);
  ctx.font="48px Arial";ctx.textAlign="center";
  ctx.fillText(vehicles[vehicle],w*(.5+carX*.25),h-35);
  if(Math.random()<.025)spawn();
  if(boost&&nitro>0)nitro=Math.max(0,nitro-1);
  else nitro=Math.min(100,nitro+.12);
  document.getElementById("nitro").textContent=Math.floor(nitro);
  requestAnimationFrame(draw);
}
document.getElementById("left").onclick=()=>carX=Math.max(-1,carX-.35);
document.getElementById("right").onclick=()=>carX=Math.min(1,carX+.35);
document.getElementById("boost").onpointerdown=()=>boost=true;
document.getElementById("boost").onpointerup=()=>boost=false;
document.getElementById("boost").onpointerleave=()=>boost=false;
document.getElementById("vehicle").onclick=()=>{
 vehicle=(vehicle+1)%vehicles.length;
 document.getElementById("status").textContent=
 "Selected: "+vehicles[vehicle];
};
draw();
</script>
</body>
</html>
