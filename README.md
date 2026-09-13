<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>YOUR PARENTS v7</title>

<style>
html,body{
    margin:0;
    overflow:hidden;
    background:#000;
    font-family:Arial;
}
canvas{display:block}
#menu,#cutscene{
position:fixed;
inset:0;
display:flex;
align-items:center;
justify-content:center;
background:rgba(0,0,0,.75);
color:white;
z-index:20;
}
#cutscene{display:none;}
#hud{
position:fixed;
inset:0;
pointer-events:none;
color:white;
}
#cross{
position:absolute;
left:50%;
top:50%;
transform:translate(-50%,-50%);
font-size:24px;
}
#hint{
position:absolute;
left:20px;
bottom:20px;
background:rgba(0,0,0,.45);
padding:10px 15px;
border-radius:10px;
}
button{
padding:12px 25px;
font-size:18px;
border:none;
border-radius:12px;
cursor:pointer;
}
</style>
</head>

<body>

<div id="menu">
<div style="text-align:center">
<h1>YOUR PARENTS v7</h1>
<button id="start">BẮT ĐẦU</button>
<p>WASD • Shift • Chuột • Space • E</p>
</div>
</div>

<div id="cutscene">
<div style="text-align:center">
<h2>Hắn phát hiện bạn!</h2>
<p>Chuẩn bị nhảy qua cửa sổ lên đoàn tàu...</p>
</div>
</div>

<div id="hud">
<div id="cross">+</div>
<div id="hint">Đi tới cánh cửa trước mặt.</div>
</div>

<script type="module">

import * as THREE from "https://cdn.jsdelivr.net/npm/three@0.170/build/three.module.js";

const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
document.body.appendChild(renderer.domElement);

const scene=new THREE.Scene();
scene.background=new THREE.Color(0xe9eef7);
scene.fog=new THREE.Fog(0xe9eef7,8,70);

const camera=new THREE.PerspectiveCamera(
78,
innerWidth/innerHeight,
0.1,
150
);

camera.position.set(0,1.7,0);

scene.add(new THREE.HemisphereLight(0xffffff,0xbca98b,2));

const sun=new THREE.DirectionalLight(0xffffff,.9);
sun.position.set(8,12,4);
scene.add(sun);

const wallMat=new THREE.MeshStandardMaterial({color:0xffffff});
const floorMat=new THREE.MeshStandardMaterial({color:0x3c2b22});

// sàn
const floor=new THREE.Mesh(
new THREE.PlaneGeometry(8,42),
floorMat
);
floor.rotation.x=-Math.PI/2;
floor.position.z=-18;
scene.add(floor);

// trần
const ceiling=new THREE.Mesh(
new THREE.PlaneGeometry(8,42),
new THREE.MeshStandardMaterial({color:0xf7f6f1})
);
ceiling.rotation.x=Math.PI/2;
ceiling.position.set(0,3,-18);
scene.add(ceiling);

// tường
[-3,3].forEach(x=>{
const w=new THREE.Mesh(
new THREE.BoxGeometry(.2,3,42),
wallMat
);
w.position.set(x,1.5,-18);
scene.add(w);
});

// cửa
const doorPivot=new THREE.Group();
doorPivot.position.set(0,0,-30);
scene.add(doorPivot);

const door=new THREE.Mesh(
new THREE.BoxGeometry(1.3,2.3,.08),
new THREE.MeshStandardMaterial({color:0xf8f8f8})
);
door.position.set(.65,1.15,0);
doorPivot.add(door);

// camera FPS
let yaw=0;
let pitch=0;
const keys={};

document.getElementById("start").onclick=()=>{
document.getElementById("menu").style.display="none";
renderer.domElement.requestPointerLock();
};

document.addEventListener("keydown",e=>keys[e.key.toLowerCase()]=true);
document.addEventListener("keyup",e=>keys[e.key.toLowerCase()]=false);

document.addEventListener("mousemove",e=>{
if(document.pointerLockElement!==renderer.domElement)return;

yaw-=e.movementX*.0025;
pitch=Math.max(-1.1,Math.min(1.1,pitch-e.movementY*.0022));
});

const forward=new THREE.Vector3();
const right=new THREE.Vector3();

let opened=false;

function animate(){

requestAnimationFrame(animate);

camera.rotation.order="YXZ";
camera.rotation.y=yaw;
camera.rotation.x=pitch;

const speed=keys.shift?0.26:0.15;

forward.set(-Math.sin(yaw),0,-Math.cos(yaw));
right.set(Math.cos(yaw),0,-Math.sin(yaw));

if(keys.w)camera.position.addScaledVector(forward,speed);
if(keys.s)camera.position.addScaledVector(forward,-speed);
if(keys.a)camera.position.addScaledVector(right,-speed);
if(keys.d)camera.position.addScaledVector(right,speed);

camera.position.x=Math.max(-2.3,Math.min(2.3,camera.position.x));
camera.position.z=Math.max(-29,Math.min(0,camera.position.z));

const nearDoor=
camera.position.distanceTo(new THREE.Vector3(0,1.7,-28.5))<2;

if(nearDoor&&!opened){
document.getElementById("hint").textContent="Nhấn E để mở cửa";
}

if(nearDoor&&keys.e&&!opened){
opened=true;
}

if(opened){
doorPivot.rotation.y=Math.max(-1.7,doorPivot.rotation.y-.05);

if(doorPivot.rotation.y<=-1.69){
document.getElementById("cutscene").style.display="flex";
document.exitPointerLock();
}
}

renderer.render(scene,camera);

}

animate();

addEventListener("resize",()=>{

camera.aspect=innerWidth/innerHeight;
camera.updateProjectionMatrix();

renderer.setSize(innerWidth,innerHeight);

});

</script>

</body>
</html>
