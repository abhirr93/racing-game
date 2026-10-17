
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>Ultimate 3D Racing</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#101827;color:white;font-family:Arial,sans-serif;overflow:hidden}
button{border:0;border-radius:12px;padding:12px;background:#26374f;color:white;font-weight:bold;font-size:14px;cursor:pointer;touch-action:manipulation}
button:disabled{opacity:.4}
#game{position:fixed;inset:0}
#hud{position:fixed;top:8px;left:8px;right:8px;display:flex;justify-content:space-between;gap:5px;pointer-events:none}
.stat{background:#101827d9;border:1px solid #ffffff33;padding:8px;border-radius:10px;font-size:12px;font-weight:bold}
#controls{position:fixed;bottom:12px;left:0;right:0;display:none;justify-content:space-between;align-items:end;padding:0 12px;pointer-events:none}
.group{display:flex;gap:7px;pointer-events:auto}
.control{width:58px;height:58px;font-size:23px;background:#101827dd;border:1px solid #ffffff77}
#boost{background:#00aaff;color:#001522;font-size:13px}
#menu{position:fixed;inset:0;overflow:auto;background:linear-gradient(#07182cbb,#07101eeF);display:flex;align-items:center;justify-content:center;padding:12px}
.panel{width:min(520px,100%);max-height:94vh;overflow:auto;background:#121f31f5;border:1px solid #ffffff30;border-radius:20px;padding:17px;text-align:center}
h1{font-size:26px;margin:5px}
p{color:#d6e2f1;font-size:14px}
.section{text-align:left;font-size:14px;font-weight:bold;margin-top:15px}
.cards{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:7px;margin-top:8px}
.card{padding:10px 4px;border:1px solid #ffffff30;background:#ffffff0b;border-radius:12px;font-size:12px;min-width:0}
.card.active{border:2px solid #ffbf28;background:#ffbf2822}
.emoji{font-size:27px;display:block;margin-bottom:5px}
.primary{width:100%;background:#ffb91f;color:#17202b;font-size:17px;margin-top:13px}
#pause{position:fixed;right:10px;top:58px;display:none}
#notice{position:fixed;left:50%;top:43%;transform:translate(-50%,-50%);font-size:32px;font-weight:900;text-align:center;text-shadow:0 3px 8px black;pointer-events:none}
.small{font-size:11px;color:#a9bdd5}
</style>
</head>
<body>
<div id="game"></div>
<div id="hud">
  <div class="stat">🏁 <span id="score">0</span></div>
  <div class="stat">🪙 <span id="coins">0</span></div>
  <div class="stat">⚡ <span id="nitroValue">100</span>%</div>
</div>
<button id="pause">Ⅱ PAUSE</button>
<div id="notice"></div>
<div id="controls">
  <div class="group">
    <button class="control" id="left">◀</button>
    <button class="control" id="right">▶</button>
  </div>
  <div class="group">
    <button class="control" id="brake">▼</button>
    <button class="control" id="boost">⚡ NITRO</button>
  </div>
</div>
<div id="menu">
  <div class="panel">
    <h1 id="heading">🏎️ ULTIMATE 3D RACING</h1>
    <p id="description">Drive, collect coins, unlock vehicles and explore new roads!</p>
    <p>🪙 Coins: <b id="wallet">0</b> &nbsp; 🏆 Best: <b id="best">0</b></p>
    <div class="section">🚘 VEHICLE GARAGE</div>
    <div class="cards" id="vehicleCards"></div>
    <div class="section">🛣️ CHOOSE YOUR ROAD</div>
    <div class="cards" id="roadCards"></div>
    <button class="primary" id="start">START RACE ▶</button>
    <p class="small">Touch controls: Left / Right • Brake • Hold Nitro</p>
    <p class="small">3D graphics need an internet connection to load Three.js.</p>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
(() => {
"use strict";
const $ = id => document.getElementById(id);
const vehicleData = [
 {name:"Starter Car",emoji:"🚗",type:"car",cost:0,color:0xe84141},
 {name:"Sport GT",emoji:"🏎️",type:"car",cost:100,color:0x168cff},
 {name:"Super Bike",emoji:"🏍️",type:"bike",cost:180,color:0xffd22f},
 {name:"Monster Truck",emoji:"🚚",type:"truck",cost:280,color:0x28c76f},
 {name:"Purple Racer",emoji:"🚙",type:"car",cost:400,color:0xb25cff},
 {name:"Turbo Bike",emoji:"🏍️",type:"bike",cost:500,color:0xff852e}
];
const roadData = [
 {name:"City",cost:0,sky:0x80c8ef,ground:0x49974c,road:0x353941,side:0x9ca3af},
 {name:"Highway",cost:50,sky:0x98c9e7,ground:0x63875a,road:0x292d35,side:0xf3d57a},
 {name:"Forest",cost:100,sky:0x80b8a5,ground:0x27613c,road:0x343a3a,side:0x1b4c2a},
 {name:"Mountain",cost:150,sky:0xa9c9e5,ground:0x8c9a85,road:0x55575c,side:0xe5e7eb},
 {name:"Desert",cost:200,sky:0xf3c88c,ground:0xcda45f,road:0x514c49,side:0xf6d58a}
];
let wallet=Number(localStorage.getItem("ur_coins")||0);
let ownedVehicles=JSON.parse(localStorage.getItem("ur_vehicles")||"[0]");
let ownedRoads=JSON.parse(localStorage.getItem("ur_roads")||"[0]");
let selectedVehicle=0, selectedRoad=0;
let scene,camera,renderer,player,roadMesh,groundMesh;
let marks=[],decor=[],traffic=[],pickups=[];
let running=false,paused=false,ended=false;
let lane=0,targetLane=0,score=0,nitro=100,spawnClock=0,pickupClock=0;
let last=0,boostHeld=false;
const input={left:false,right:false,brake:false};
const bestKey="ur_best";
function persist(){
 localStorage.setItem("ur_coins",wallet);
 localStorage.setItem("ur_vehicles",JSON.stringify(ownedVehicles));
 localStorage.setItem("ur_roads",JSON.stringify(ownedRoads));
}
function moneyUI(){
 $("coins").textContent=wallet;$("wallet").textContent=wallet;
 $("best").textContent=localStorage.getItem(bestKey)||"0";
}
function makeCards(){
 const vc=$("vehicleCards");vc.innerHTML="";
 vehicleData.forEach((v,i)=>{
  const b=document.createElement("button");
  const owned=ownedVehicles.includes(i);
  b.className="card"+(selectedVehicle===i?" active":"");
  b.innerHTML=`<span class="emoji">${v.emoji}</span><b>${v.name}</b><br><br>${owned?"OWNED":"🪙 "+v.cost}`;
  b.onclick=()=>{
   if(!ownedVehicles.includes(i)){
    if(wallet<v.cost){$("description").textContent="Not enough coins! Collect more coins during races.";return;}
    wallet-=v.cost;ownedVehicles.push(i);persist();
   }
   selectedVehicle=i;makeCards();moneyUI();
  };
  vc.appendChild(b);
 });
 const rc=$("roadCards");rc.innerHTML="";
 roadData.forEach((r,i)=>{
  const b=document.createElement("button");
  const owned=ownedRoads.includes(i);
  b.className="card"+(selectedRoad===i?" active":"");
  b.innerHTML=`<span class="emoji">${["🏙️","🛣️","🌲","⛰️","🏜️"][i]}</span><b>${r.name}</b><br><br>${owned?"UNLOCKED":"🪙 "+r.cost}`;
  b.onclick=()=>{
   if(!ownedRoads.includes(i)){
    if(wallet<r.cost){$("description").textContent="Not enough coins to unlock this road. Keep racing!";return;}
    wallet-=r.cost;ownedRoads.push(i);persist();
   }
   selectedRoad=i;makeCards();moneyUI();
  };
  rc.appendChild(b);
 });
 moneyUI();
}
function material(color,roughness=.8){
 return new THREE.MeshStandardMaterial({color,roughness});
}
function meshBox(parent,w,h,d,color,x,y,z){
 const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d),material(color));
 m.position.set(x,y,z);parent.add(m);return m;
}
function createVehicle(data){
 const g=new THREE.Group(), c=data.color;
 if(data.type==="bike"){
  const tireMat=material(0x16191d);
  for(const z of [-.68,.62]){
   const wheel=new THREE.Mesh(new THREE.TorusGeometry(.25,.09,8,16),tireMat);
   wheel.rotation.y=Math.PI/2;wheel.position.set(0,.28,z);g.add(wheel);
  }
  meshBox(g,.25,.3,.72,c,0,.58,0);
  meshBox(g,.65,.13,.26,0x252a32,0,.83,-.05);
  meshBox(g,.12,.24,.18,0xddeeff,0,.87,.45);
  meshBox(g,.55,.08,.12,0x333333,0,.65,-.35);
 }else{
  const truck=data.type==="truck";
  meshBox(g,truck?1.55:1.22,truck?.65:.38,truck?2.5:2.05,c,0,truck?.58:.48,0);
  if(truck){
   meshBox(g,1.35,.8,1.05,0xe6edf4,0,1.2,.35);
   meshBox(g,1.37,.07,.08,0x343b45,0,1.18,.89);
  }else{
   meshBox(g,.85,.43,.94,0x92d8f5,0,.86,-.13);
   meshBox(g,1.02,.12,.45,c,0,.67,.68);
  }
  const wheelMat=material(0x16191d);
  const wheelR=truck?.31:.23;
  for(const x of [-.66,.66])for(const z of [-.65,.65]){
   const wheel=new THREE.Mesh(new THREE.CylinderGeometry(wheelR,wheelR,.15,16),wheelMat);
   wheel.rotation.z=Math.PI/2;wheel.position.set(x,wheelR,z);g.add(wheel);
  }
  for(const x of [-.38,.38]){
   meshBox(g,.2,.1,.04,0xfff3c4,x,.49,truck?1.26:1.04);
  }
 }
 return g;
}
function createScene(){
 scene=new THREE.Scene();
 const rd=roadData[selectedRoad];
 scene.background=new THREE.Color(rd.sky);
 scene.fog=new THREE.Fog(rd.sky,50,170);
 camera=new THREE.PerspectiveCamera(65,innerWidth/innerHeight,.1,350);
 camera.position.set(0,5.6,10);
 renderer=new THREE.WebGLRenderer({antialias:true});
 renderer.setPixelRatio(Math.min(devicePixelRatio,1.7));
 renderer.setSize(innerWidth,innerHeight);
 renderer.shadowMap.enabled=true;
 $("game").innerHTML="";
 $("game").appendChild(renderer.domElement);
 scene.add(new THREE.HemisphereLight(0xe9f5ff,rd.ground,2));
 const sun=new THREE.DirectionalLight(0xffffff,2);
 sun.position.set(-8,15,8);sun.castShadow=true;scene.add(sun);
 groundMesh=new THREE.Mesh(new THREE.PlaneGeometry(240,330),material(rd.ground));
 groundMesh.rotation.x=-Math.PI/2;groundMesh.position.set(0,-.16,-100);scene.add(groundMesh);
 roadMesh=new THREE.Mesh(new THREE.PlaneGeometry(10,330),material(rd.road));
 roadMesh.rotation.x=-Math.PI/2;roadMesh.position.set(0,-.06,-100);scene.add(roadMesh);
 marks=[];decor=[];traffic=[];pickups=[];
 for(let z=-260;z<30;z+=9){
  const m=meshBox(scene,.12,.025,3.8,0xffffff,0,.01,z);marks.push(m);
 }
 for(let z=-260;z<30;z+=18){
  for(const x of [-1.67,1.67]){
   marks.push(meshBox(scene,.075,.025,2.8,rd.side,x,.015,z));
  }
 }
 for(let i=0;i<52;i++){
  const side=i%2===0?-1:1;
  const x=side*(7+Math.random()*9),z=-Math.random()*270;
  const group=new THREE.Group();
  group.position.set(x,0,z);
  if(selectedRoad===0){
   meshBox(group,1.1,2.4,1.1,0xd5b28a,0,1.2,0);
   meshBox(group,1.2,1.2,1.2,[0x4d9eec,0xe6a45c,0xc8d5e5][i%3],0,3,0);
  }else if(selectedRoad===1){
   meshBox(group,.15,2.6,.15,0x656b75,0,1.3,0);
   meshBox(group,.55,.16,.55,0xfff3bc,0,2.65,0);
  }else if(selectedRoad===2){
   meshBox(group,.35,1.5,.35,0x684a2c,0,.75,0);
   const crown=new THREE.Mesh(new THREE.ConeGeometry(1.5,3,7),material(0x24703a));
   crown.position.y=2.5;group.add(crown);
  }else if(selectedRoad===3){
   const rock=new THREE.Mesh(new THREE.ConeGeometry(3,5,5),material(0x858c91));
   rock.position.y=2.2;group.add(rock);
   meshBox(group,3,.35,3,0xf0f3f5,0,4.1,0);
  }else{
   meshBox(group,.25,1.2,.25,0x80582e,0,.6,0);
   const palm=new THREE.Mesh(new THREE.ConeGeometry(1.3,.3,6),material(0x4c8038));
   palm.rotation.x=Math.PI/2;palm.position.y=1.3;group.add(palm);
  }
  scene.add(group);decor.push(group);
 }
 player=createVehicle(vehicleData[selectedVehicle]);
 player.position.set(0,0,3);scene.add(player);
}
function clearObjects(){
 for(const a of [...traffic,...pickups])scene.remove(a.mesh);
 traffic=[];pickups=[];
}
function spawnTraffic(){
 const x=[-3,0,3][Math.floor(Math.random()*3)];
 const data={...vehicleData[Math.floor(Math.random()*vehicleData.length)],color:[0xe34d4d,0xf4f4f4,0x202a3b,0x27b3a2][Math.floor(Math.random()*4)]};
 const mesh=createVehicle(data);mesh.scale.set(.88,.88,.88);
 mesh.position.set(x,0,-105);scene.add(mesh);
 traffic.push({mesh,x,z:-105});
}
function spawnPickup(){
 const x=[-3,0,3][Math.floor(Math.random()*3)];
 const coin=Math.random()<.76;
 const mesh=new THREE.Mesh(
  coin?new THREE.TorusGeometry(.4,.12,8,18):new THREE.OctahedronGeometry(.48),
  new THREE.MeshStandardMaterial({color:coin?0xffcf28:0x00baff,emissive:coin?0x5a3700:0x004d99,metalness:.4,roughness:.25})
 );
 mesh.position.set(x,1.1,-100);scene.add(mesh);
 pickups.push({mesh,x,z:-100,coin});
}
function updateRoad(){
 const rd=roadData[selectedRoad];
 scene.background.setHex(rd.sky);scene.fog.color.setHex(rd.sky);
 groundMesh.material.color.setHex(rd.ground);
 roadMesh.material.color.setHex(rd.road);
 marks.forEach(m=>{
  m.position.z+=speed()*2.4;
  if(m.position.z>15)m.position.z-=290;
 });
 decor.forEach(g=>{
  g.position.z+=speed()*2.4;
  if(g.position.z>15)g.position.z=-255-Math.random()*20;
 });
}
function speed(){return input.brake?.28:(boostHeld&&nitro>0?.98:.57);}
function startGame(){
 if(typeof THREE==="undefined"){
  $("description").textContent="3D engine did not load. Check your internet connection and refresh.";
  return;
 }
 if(renderer){renderer.dispose();$("game").innerHTML="";}
 createScene();clearObjects();
 score=0;nitro=100;lane=0;targetLane=0;spawnClock=0;pickupClock=0;
 running=true;paused=false;ended=false;
 $("menu").style.display="none";$("controls").style.display="flex";
 $("pause").style.display="block";$("notice").textContent="";
 $("score").textContent="0";$("nitroValue").textContent="100";
 last=performance.now();requestAnimationFrame(loop);
}
function endGame(){
 if(!running)return;
 running=false;ended=true;
 $("controls").style.display="none";$("pause").style.display="none";
 const reward=Math.floor(score/100)+3;
 wallet+=reward;
 const best=Math.max(score,Number(localStorage.getItem(bestKey)||0));
 localStorage.setItem(bestKey,best);persist();moneyUI();makeCards();
 $("heading").textContent="🏁 RACE OVER!";
 $("description").textContent=`Score ${score} • Reward +${reward} coins. Choose a vehicle or road and race again!`;
 $("start").textContent="RACE AGAIN ▶";$("menu").style.display="flex";
}
function togglePause(){
 if(!running&&!paused)return;
 paused=!paused;
 if(paused){$("pause").textContent="▶ RESUME";$("notice").textContent="PAUSED";}
 else{$("pause").textContent="Ⅱ PAUSE";$("notice").textContent="";last=performance.now();requestAnimationFrame(loop);}
}
function loop(now){
 if(!running||paused)return;
 const dt=Math.min((now-last)/16.67,2);last=now;
 if(input.left)targetLane=Math.max(-3,targetLane-.12*dt);
 if(input.right)targetLane=Math.min(3,targetLane+.12*dt);
 lane+=(targetLane-lane)*.17*dt;
 player.position.x=lane;
 player.rotation.z=(targetLane-lane)*-.05;
 camera.position.x+=(lane*.23-camera.position.x)*.05*dt;
 camera.lookAt(lane*.12,.35,-1);
 if(boostHeld&&nitro>0)nitro=Math.max(0,nitro-.8*dt);
 else nitro=Math.min(100,nitro+.17*dt);
 updateRoad();
 spawnClock+=dt;pickupClock+=dt;
 if(spawnClock>Math.max(30,70-score*.015)){spawnTraffic();spawnClock=0;}
 if(pickupClock>42){spawnPickup();pickupClock=0;}
 for(const t of traffic){
  t.z+=speed()*2.4*dt;t.mesh.position.z=t.z;
  if(Math.abs(t.z-player.position.z)<1.5&&Math.abs(t.x-lane)<1.2){endGame();return;}
  if(t.z>6)score+=.03*dt;
 }
 for(const p of pickups){
  p.z+=speed()*2.4*dt;p.mesh.position.z=p.z;p.mesh.rotation.y+=.07*dt;
  if(Math.abs(p.z-player.position.z)<1.7&&Math.abs(p.x-lane)<1.3){
   if(p.coin){wallet+=5;persist();moneyUI();}
   else nitro=Math.min(100,nitro+30);
   p.z=40;
  }
 }
 traffic=traffic.filter(t=>{if(t.z>18){scene.remove(t.mesh);return false;}return true;});
 pickups=pickups.filter(p=>{if(p.z>18){scene.remove(p.mesh);return false;}return true;});
 score+=speed()*.65*dt;
 $("score").textContent=Math.floor(score);
 $("nitroValue").textContent=Math.floor(nitro);
 renderer.render(scene,camera);
 if(running&&!paused)requestAnimationFrame(loop);
}
function hold(id,down,up){
 const el=$(id);
 el.addEventListener("pointerdown",e=>{e.preventDefault();down();});
 ["pointerup","pointercancel","pointerleave"].forEach(ev=>el.addEventListener(ev,e=>{up();}));
}
hold("left",()=>input.left=true,()=>input.left=false);
hold("right",()=>input.right=true,()=>input.right=false);
hold("brake",()=>input.brake=true,()=>input.brake=false);
hold("boost",()=>boostHeld=true,()=>boostHeld=false);
$("start").onclick=()=>{
 $("heading").textContent="🏎️ ULTIMATE 3D RACING";
 $("start").textContent="START RACE ▶";
 startGame();
};
$("pause").onclick=togglePause;
addEventListener("keydown",e=>{
 if(e.key==="ArrowLeft"||e.key==="a")input.left=true;
 if(e.key==="ArrowRight"||e.key==="d")input.right=true;
 if(e.key==="ArrowDown")input.brake=true;
 if(e.key===" "||e.key==="Shift")boostHeld=true;
});
addEventListener("keyup",e=>{
 if(e.key==="ArrowLeft"||e.key==="a")input.left=false;
 if(e.key==="ArrowRight"||e.key==="d")input.right=false;
 if(e.key==="ArrowDown")input.brake=false;
 if(e.key===" "||e.key==="Shift")boostHeld=false;
});
addEventListener("resize",()=>{
 if(renderer&&camera){
  camera.aspect=innerWidth/innerHeight;camera.updateProjectionMatrix();
  renderer.setSize(innerWidth,innerHeight);
 }
});
moneyUI();makeCards();
})();
</script>
</body>
</html>
