<!doctype html>
<html lang="ar">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>زهرة الجواء — 3D Drive-Thru Design</title>
<style>
  html,body { height:100%; margin:0; font-family: Arial, sans-serif; direction: rtl; }
  #container { width:100%; height:100%; display:block; position:relative; overflow:hidden;}
  /* جدول الأبعاد الجانبي */
  #infoPanel {
    position: absolute;
    top: 12px;
    left: 12px;                   /* لأن الصفحة بالاتجاه RTL، لكن لوحة الأبعاد باللغة الانجليزية */
    background: rgba(255,255,255,0.95);
    border-radius:6px;
    padding:12px;
    box-shadow:0 6px 18px rgba(0,0,0,0.25);
    font-size:14px;
    color:#111;
    min-width:170px;
  }
  #infoPanel h3 { margin:0 0 8px 0; font-size:16px; text-align:left; }
  table { width:100%; border-collapse:collapse; font-size:13px; }
  td, th { padding:6px 4px; text-align:left; }
  th { font-weight:700; font-size:12px; color:#444; }
  /* تعليمات صغيرة في الأسفل */
  #hint {
    position:absolute; bottom:10px; right:12px;
    background:rgba(255,255,255,0.9); padding:8px 10px; border-radius:6px;
    font-size:12px; color:#333;
  }
  /* اسم اللوحة على واجهة المبنى — هذا مجرد عنصرمكان؛ النص يظهر فعليًا داخل المشهد كـ texture */
  @media (max-width:600px) {
    #infoPanel { font-size:12px; min-width:150px; padding:8px; }
  }
</style>
</head>
<body>
<div id="container"></div>

<div id="infoPanel" aria-hidden="true">
  <h3>Dimensions (English)</h3>
  <table>
    <tr><th>Parameter</th><th>Measurement</th></tr>
    <tr><td>Building Length</td><td>8.5 m</td></tr>
    <tr><td>Building Width</td><td>4.5 m</td></tr>
    <tr><td>Street Width</td><td>7 m</td></tr>
    <tr><td>Drive-Thru Lane</td><td>3 m</td></tr>
    <tr><td>Garden Width</td><td>4.5 m</td></tr>
  </table>
</div>

<div id="hint">Use mouse / touch: rotate, pan, zoom</div>

<!-- Three.js from CDN -->
<script type="module">
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.module.js';
import { OrbitControls } from 'https://cdn.jsdelivr.net/npm/three@0.152.2/examples/jsm/controls/OrbitControls.js';
import { GLTFExporter } from 'https://cdn.jsdelivr.net/npm/three@0.152.2/examples/jsm/exporters/GLTFExporter.js';

const container = document.getElementById('container');
const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.outputColorSpace = THREE.SRGBColorSpace;
container.appendChild(renderer.domElement);

const scene = new THREE.Scene();
scene.background = new THREE.Color(0xbfe9ff); // sky color (day)
scene.fog = new THREE.Fog(0xbfe9ff, 80, 180);

// Camera: top-ish perspective
const camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight, 0.1, 500);
camera.position.set(0, 35, 45); // high and back
camera.lookAt(0,0,0);

// Controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.target.set(0,0,0);
controls.maxPolarAngle = Math.PI/2 - 0.1;
controls.minDistance = 5;
controls.maxDistance = 120;
controls.update();

// Lights (day)
const hemi = new THREE.HemisphereLight(0xffffff, 0x444444, 0.9);
hemi.position.set(0,50,0);
scene.add(hemi);

const dir = new THREE.DirectionalLight(0xffffff, 0.9);
dir.position.set(30,40,10);
dir.castShadow = true;
dir.shadow.mapSize.set(2048,2048);
scene.add(dir);

// Ground (street + sidewalks)
const ground = new THREE.Group();
scene.add(ground);

// Parameters (in meters -> 1 unit = 1 meter)
const buildingLength = 8.5;  // along X (street direction)
const buildingWidth = 4.5;   // depth from street into site (Z)
const buildingHeight = 6;    // two floors ~ 3m each
const roofOverhang = 0.2;
const streetWidth = 7;       // two lanes (3.5m each approx.) - we will use 7m
const driveLaneWidth = 3;    // width of drive-thru lane
const gardenWidth = buildingWidth; // garden depth equals building width
const siteDepth = buildingWidth + 6; // enough rear space for garden & back
const siteLength = buildingLength + 12; // extra space front and sides

// create asphalt road (big plane)
const roadGeo = new THREE.PlaneGeometry(siteLength, streetWidth + 12);
const roadMat = new THREE.MeshStandardMaterial({ color:0x333333 });
const road = new THREE.Mesh(roadGeo, roadMat);
road.rotation.x = -Math.PI/2;
road.receiveShadow = true;
road.position.set(0, -0.001, (streetWidth/2) - 1); // slightly offset so building aligns
ground.add(road);

// lane markings (simple thin planes)
function addMark(x,z,w,l) {
  const g = new THREE.PlaneGeometry(l, w);
  const m = new THREE.MeshBasicMaterial({ color:0xffffff });
  const p = new THREE.Mesh(g,m);
  p.rotation.x = -Math.PI/2;
  p.position.set(x, 0.01, z);
  ground.add(p);
}
// add center dashed line (approx)
for (let i=-40; i<=40; i+=6){
  addMark(i, road.position.z - (streetWidth/4), 0.15, 2.8);
  addMark(i, road.position.z + (streetWidth/4), 0.15, 2.8);
}

// Sidewalks (lighter)
const sidewalkGeo = new THREE.PlaneGeometry(siteLength, 3.2);
const sidewalkMat = new THREE.MeshStandardMaterial({ color:0xd9d9d9 });
const sidewalk = new THREE.Mesh(sidewalkGeo, sidewalkMat);
sidewalk.rotation.x = -Math.PI/2;
sidewalk.position.set(0, 0.001, road.position.z + (streetWidth/2) + 1.6);
ground.add(sidewalk);

// Drive-thru path (curve / lane) - represented by a thin lighter asphalt path running in front of building
const laneGroup = new THREE.Group();
ground.add(laneGroup);

// create a curved drive lane using tubular approach: approximate with path of rectangles
const laneMaterial = new THREE.MeshStandardMaterial({ color:0x2b2b2b });
const laneLeftX = - (buildingLength/2) - 4; // entrance from right -> go left along front of building
// We'll create a rectangular drive lane area (simplified)
const driveLaneGeo = new THREE.PlaneGeometry(buildingLength + 10, driveLaneWidth);
const driveLane = new THREE.Mesh(driveLaneGeo, laneMaterial);
driveLane.rotation.x = -Math.PI/2;
driveLane.position.set(0, 0.002, sidewalk.position.z - 2.2); // in front of building
laneGroup.add(driveLane);

// lane border lines
const borderGeo = new THREE.PlaneGeometry(buildingLength + 10, 0.05);
const borderMat = new THREE.MeshBasicMaterial({ color:0xffffff });
const b1 = new THREE.Mesh(borderGeo, borderMat);
b1.rotation.x = -Math.PI/2;
b1.position.set(0,0.003, driveLane.position.z - (driveLaneWidth/2) + 0.025);
laneGroup.add(b1);
const b2 = b1.clone();
b2.position.set(0,0.003, driveLane.position.z + (driveLaneWidth/2) - 0.025);
laneGroup.add(b2);

// Building base (rectangle)
const buildingGroup = new THREE.Group();
scene.add(buildingGroup);

// Place building so its front aligns to the drive lane and street
const buildingFrontZ = driveLane.position.z + 0.9; // a bit toward sidewalk
const buildingCenterX = 0; // center along X axis
const buildingCenterZ = buildingFrontZ - (buildingWidth/2);

// main box (two floors)
const boxGeo = new THREE.BoxGeometry(buildingLength, buildingHeight, buildingWidth);
const boxMat = new THREE.MeshStandardMaterial({ color: 0xf2efe6 });
const box = new THREE.Mesh(boxGeo, boxMat);
box.castShadow = true;
box.receiveShadow = true;
box.position.set(buildingCenterX, buildingHeight/2, buildingCenterZ);
buildingGroup.add(box);

// Glass facade (front)
const glassHeight = buildingHeight - 0.2;
const glassGeo = new THREE.PlaneGeometry(buildingLength - 0.2, glassHeight);
const glassMat = new THREE.MeshPhysicalMaterial({
  color: 0x88cfe8, metalness:0.1, roughness:0.05, transmission:0.9, transparent:true, opacity:0.95
});
const glass = new THREE.Mesh(glassGeo, glassMat);
glass.position.set(buildingCenterX, glassHeight/2 + 0.05, buildingFrontZ + 0.01);
buildingGroup.add(glass);

// Create a canvas texture for Arabic sign "زهرة الجواء"
function makeSignTexture(textArabic, textEnglish=null) {
  const w = 1024, h = 256;
  const canvas = document.createElement('canvas');
  canvas.width = w; canvas.height = h;
  const ctx = canvas.getContext('2d');
  // background semi-transparent
  ctx.fillStyle = 'rgba(255,255,255,0.02)';
  ctx.fillRect(0,0,w,h);
  // Arabic text (right-to-left)
  ctx.fillStyle = '#0b2b3a';
  // choose font that usually supports Arabic on most systems; fallback to serif
  ctx.font = 'bold 86px "Scheherazade", "Arial", sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  // draw Arabic
  ctx.fillText(textArabic, w/2, h/2 - 8);
  // optional English under it
  if (textEnglish) {
    ctx.font = 'bold 28px Arial';
    ctx.fillText(textEnglish, w/2, h - 32);
  }
  const tex = new THREE.CanvasTexture(canvas);
  tex.encoding = THREE.sRGBEncoding;
  tex.needsUpdate = true;
  return tex;
}
const signTex = makeSignTexture('زهرة الجواء', 'Zahra Al-Jawaa');
const signMat = new THREE.MeshBasicMaterial({ map: signTex, transparent:true });
const signGeo = new THREE.PlaneGeometry(Math.max(6.5, buildingLength - 1), 0.9);
const signMesh = new THREE.Mesh(signGeo, signMat);
signMesh.position.set(buildingCenterX, glassHeight - 0.4, buildingFrontZ + 0.02);
buildingGroup.add(signMesh);

// Roof - gable (triangular prism)
function makeGableRoof(length, width, height, color) {
  // create geometry manually
  const geom = new THREE.BufferGeometry();
  // 6 vertices for two triangles along length
  // we'll build as extruded triangular cross-section along X
  const halfW = width / 2;
  const h = height;
  // vertices: along X from -length/2 to +length/2, each cross has 3 verts
  const verts = new Float32Array([
    // left end (-length/2)
    -length/2, h, 0,       // top ridge
    -length/2, 0, -halfW,  // left eave
    -length/2, 0, halfW,   // right eave
    // right end (+length/2)
     length/2, h, 0,
     length/2, 0, -halfW,
     length/2, 0, halfW
  ]);
  // indices to build faces (sides and top/bottom)
  const indices = [
    // front triangle (left end)
    0,1,2,
    // back triangle (right end)
    3,5,4,
    // side faces (rectangles as two triangles)
    0,3,1,
    3,4,1,
    0,2,3,
    3,2,5,
    1,4,2,
    4,5,2
  ];
  geom.setAttribute('position', new THREE.BufferAttribute(verts,3));
  geom.setIndex(indices);
  geom.computeVertexNormals();
  const mat = new THREE.MeshStandardMaterial({ color: color });
  const mesh = new THREE.Mesh(geom, mat);
  mesh.castShadow = true;
  return mesh;
}
const roofHeight = 1.2;
const roof = makeGableRoof(buildingLength + roofOverhang*2, buildingWidth + roofOverhang*2, roofHeight, 0xc22b2b);
roof.position.set(buildingCenterX, buildingHeight + 0.01, buildingCenterZ);
buildingGroup.add(roof);

// small base curb between building and drive lane
const curbGeo = new THREE.BoxGeometry(buildingLength + 0.2, 0.2, 0.3);
const curbMat = new THREE.MeshStandardMaterial({ color:0x555555 });
const curb = new THREE.Mesh(curbGeo, curbMat);
curb.position.set(buildingCenterX, 0.1, buildingFrontZ - 0.15);
ground.add(curb);

// Garden behind building (on the side right of driver when in lane -> in our coordinate, place to building's right side along +X)
const gardenGroup = new THREE.Group();
scene.add(gardenGroup);

// place garden to the right side along X direction (length along X).
// user wanted "الحديقة بطول المبنى وهي في واجهة المبنى" — garden extends along full length, located at right of drive lane on right of driver.
// We'll place garden parallel to building on the side closer to positive X.
const gardenLength = buildingLength;
const gardenDepth = gardenWidth; // equals buildingWidth
const gardenGeo = new THREE.PlaneGeometry(gardenLength, gardenDepth);
const gardenMat = new THREE.MeshStandardMaterial({ color:0x7fc97f });
const garden = new THREE.Mesh(gardenGeo, gardenMat);
garden.rotation.x = -Math.PI/2;
garden.position.set(buildingCenterX + (buildingLength/2) - (gardenLength/2), 0.001, buildingCenterZ + (buildingWidth/2) + (gardenDepth/2) + 0.4);
gardenGroup.add(garden);

// add some simple trees (cylinders + spheres)
function addTree(x,z,scale=1) {
  const trunkGeo = new THREE.CylinderGeometry(0.08*scale,0.08*scale,0.6*scale,8);
  const trunkMat = new THREE.MeshStandardMaterial({ color:0x7a4a2a });
  const trunk = new THREE.Mesh(trunkGeo, trunkMat);
  trunk.position.set(x, 0.3*scale, z);
  gardenGroup.add(trunk);
  const crownGeo = new THREE.SphereGeometry(0.6*scale, 12, 12);
  const crownMat = new THREE.MeshStandardMaterial({ color:0x1f8b2e });
  const crown = new THREE.Mesh(crownGeo, crownMat);
  crown.position.set(x, 0.9*scale, z);
  gardenGroup.add(crown);
}
// scatter a few trees along garden length
for (let i=-buildingLength/2 + 1; i<=buildingLength/2 - 1; i += 2.4) {
  addTree(i, garden.position.z, 1.0);
}

// Add a simple car (box) to show drive-thru position (left of driver -> car passes with driver's side left)
function addCar(x,z,rot=0, color=0xffcc00) {
  const car = new THREE.Group();
  const body = new THREE.BoxGeometry(1.7, 0.5, 0.9);
  const mat = new THREE.MeshStandardMaterial({ color: color });
  const bodyMesh = new THREE.Mesh(body, mat);
  bodyMesh.position.set(0, 0.35, 0);
  car.add(bodyMesh);
  const wheelGeo = new THREE.CylinderGeometry(0.15,0.15,0.08,12);
  const wheelMat = new THREE.MeshStandardMaterial({ color:0x111111 });
  const wheelPositions = [
    [-0.6, 0.15, -0.4],
    [0.6, 0.15, -0.4],
    [-0.6, 0.15, 0.4],
    [0.6, 0.15, 0.4]
  ];
  wheelPositions.forEach(wp => {
    const w = new THREE.Mesh(wheelGeo, wheelMat);
    w.rotation.z = Math.PI/2;
    w.position.set(wp[0], wp[1], wp[2]);
    car.add(w);
  });
  car.position.set(x, 0, z);
  car.rotation.y = rot;
  scene.add(car);
  return car;
}
// place one car in drive lane (at pickup window, which we'll place near center front)
const pickupX = -1.0;
const pickupZ = driveLane.position.z;
addCar(pickupX, pickupZ, Math.PI/2, 0xff4d4d);

// Minor extras: signage posts and arrows on pavement
function addArrow(x,z,rot=0) {
  const aGeo = new THREE.PlaneGeometry(2.2, 1.0);
  const aMat = new THREE.MeshBasicMaterial({ color:0xffffff, side:THREE.DoubleSide });
  const a = new THREE.Mesh(aGeo, aMat);
  // use canvas texture to draw arrow
  const c = document.createElement('canvas');
  c.width=512; c.height=256;
  const ctx = c.getContext('2d');
  ctx.fillStyle='rgba(255,255,255,0)';
  ctx.fillRect(0,0,512,256);
  ctx.fillStyle='#ffffff';
  ctx.beginPath();
  ctx.moveTo(256,32);
  ctx.lineTo(480,224);
  ctx.lineTo(256,160);
  ctx.lineTo(32,224);
  ctx.closePath();
  ctx.fill();
  const tx = new THREE.CanvasTexture(c);
  a.material = new THREE.MeshBasicMaterial({ map:tx, transparent:true });
  a.rotation.x = -Math.PI/2;
  a.position.set(x, 0.02, z);
  a.rotation.z = rot;
  ground.add(a);
}
addArrow( -6, driveLane.position.z, 0);

// small wall between garden and drive lane
const wallGeo = new THREE.BoxGeometry(gardenLength + 0.4, 0.4, 0.12);
const wallMat = new THREE.MeshStandardMaterial({ color:0xaaaaaa });
const wall = new THREE.Mesh(wallGeo, wallMat);
wall.position.set(garden.position.x, 0.2, garden.position.z - gardenDepth/2 - 0.06);
scene.add(wall);

// Shadows
renderer.shadowMap.enabled = true;
box.castShadow = true;
glass.castShadow = false;
roof.castShadow = true;
road.receiveShadow = true;

// Axes helper (optional for debugging) - commented out for presentation
// const axes = new THREE.AxesHelper(5); scene.add(axes);

// small grid ground guide
const grid = new THREE.GridHelper(60, 60, 0xcccccc, 0xeeeeee);
grid.position.y = 0.001;
scene.add(grid);

// Animation loop
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();

// resize handling
window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
});

// Optional: export GLB button (uncomment to use)
// To export the scene (download .glb), user can enable this function if desired
window.exportGLB = function() {
  const exporter = new GLTFExporter();
  exporter.parse(scene, function(result) {
    let output;
    if (result instanceof ArrayBuffer) {
      output = result;
    } else {
      output = JSON.stringify(result, null, 2);
    }
    saveArrayBuffer(output, 'zahra_drive_thru.glb');
  }, { binary: true });
};
function saveArrayBuffer(buffer, filename) {
  const blob = new Blob([buffer], { type: 'application/octet-stream' });
  const link = document.createElement('a');
  link.href = URL.createObjectURL(blob);
  link.download = filename;
  link.click();
}

// END of module script
</script>
</body>
</html>
