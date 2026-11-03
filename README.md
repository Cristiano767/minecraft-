# minecraft-
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BlockCraft</title>
 
<style>
body {
    margin: 0;
    overflow: hidden;
    font-family: Arial;
    background: #87ceeb;
}
 
#info {
    position: fixed;
    top: 15px;
    left: 15px;
    color: white;
    background: #0008;
    padding: 12px;
    border-radius: 8px;
    z-index: 2;
}
 
#crosshair {
    position: fixed;
    left: 50%;
    top: 50%;
    color: white;
    font-size: 25px;
    transform: translate(-50%, -50%);
    pointer-events: none;
}
 
#hotbar {
    position: fixed;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 8px;
    z-index: 2;
}
 
.slot {
    width: 48px;
    height: 48px;
    border: 3px solid white;
    background: #5559;
    display: grid;
    place-items: center;
    font-size: 25px;
}
 
.slot.active {
    border-color: yellow;
}
</style>
</head>
 
<body>
 
<div id="info">
    <b>BLOCKCRAFT</b><br>
    WASD = Bewegen<br>
    Maus = Umschauen<br>
    Linksklick = Abbauen<br>
    Rechtsklick = Platzieren<br>
    1–5 = Block auswählen
</div>
 
<div id="crosshair">+</div>
 
<div id="hotbar">
    <div class="slot active">🟩</div>
    <div class="slot">🟫</div>
    <div class="slot">⬜</div>
    <div class="slot">🪵</div>
    <div class="slot">🪨</div>
</div>
 
<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';
 
import { PointerLockControls } from
'https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/controls/PointerLockControls.js';
 
// Szene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);
 
const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
);
 
camera.position.set(0, 3, 8);
 
// Renderer
const renderer = new THREE.WebGLRenderer({
    antialias: true
});
 
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
document.body.appendChild(renderer.domElement);
 
// Licht
const light = new THREE.DirectionalLight(0xffffff, 2);
light.position.set(10, 20, 10);
scene.add(light);
 
scene.add(new THREE.AmbientLight(0xffffff, 1));
 
// Steuerung
const controls = new PointerLockControls(camera, renderer.domElement);
 
renderer.domElement.addEventListener('click', () => {
    controls.lock();
});
 
// Materialien
const materials = [
    new THREE.MeshLambertMaterial({color: 0x55aa33}),
    new THREE.MeshLambertMaterial({color: 0x88552a}),
    new THREE.MeshLambertMaterial({color: 0xaaaaaa}),
    new THREE.MeshLambertMaterial({color: 0x996633}),
    new THREE.MeshLambertMaterial({color: 0x555555})
];
 
// Blockwelt
const blocks = [];
const geometry = new THREE.BoxGeometry(1, 1, 1);
 
function addBlock(x, y, z, type = 0) {
    const block = new THREE.Mesh(geometry, materials[type]);
 
    block.position.set(x, y, z);
    block.userData.type = type;
 
    scene.add(block);
    blocks.push(block);
 
    return block;
}
 
// Terrain generieren
for (let x = -15; x <= 15; x++) {
    for (let z = -15; z <= 15; z++) {
 
        const height = Math.floor(
            2 + Math.sin(x * 0.3) * 1.5 +
            Math.cos(z * 0.3) * 1.5
        );
 
        for (let y = 0; y <= height; y++) {
            addBlock(x, y, z, y === height ? 0 : 1);
        }
    }
}
 
// Bäume
function createTree(x, y, z) {
    for (let i = 0; i < 3; i++) {
        addBlock(x, y + i, z, 3);
    }
 
    for (let dx = -1; dx <= 1; dx++) {
        for (let dz = -1; dz <= 1; dz++) {
            addBlock(x + dx, y + 3, z + dz, 0);
        }
    }
}
 
createTree(4, 3, 4);
createTree(-5, 3, -3);
createTree(8, 2, -7);
 
// Bewegung
const keys = {};
 
document.addEventListener('keydown', e => {
    keys[e.code] = true;
 
    if (e.key >= '1' && e.key <= '5') {
        selectedBlock = Number(e.key) - 1;
        updateHotbar();
    }
});
 
document.addEventListener('keyup', e => {
    keys[e.code] = false;
});
 
// Blockauswahl
let selectedBlock = 0;
 
function updateHotbar() {
    document.querySelectorAll('.slot').forEach((slot, i) => {
        slot.classList.toggle('active', i === selectedBlock);
    });
}
 
// Raycasting
const raycaster = new THREE.Raycaster();
raycaster.far = 6;
 
function getTarget() {
    raycaster.setFromCamera(
        new THREE.Vector2(0, 0),
        camera
    );
 
    const hits = raycaster.intersectObjects(blocks);
 
    return hits.length ? hits[0] : null;
}
 
// Abbauen
document.addEventListener('mousedown', e => {
    if (!controls.isLocked) return;
 
    const hit = getTarget();
    if (!hit) return;
 
    if (e.button === 0) {
        scene.remove(hit.object);
 
        const index = blocks.indexOf(hit.object);
        if (index !== -1) blocks.splice(index, 1);
    }
 
    if (e.button === 2) {
        const normal = hit.face.normal.clone();
 
        const position = hit.object.position.clone()
            .add(normal);
 
        const occupied = blocks.some(block =>
            block.position.distanceTo(position) < 0.1
        );
 
        if (!occupied) {
            addBlock(
                position.x,
                position.y,
                position.z,
                selectedBlock
            );
        }
    }
});
 
document.addEventListener('contextmenu', e => {
    e.preventDefault();
});
 
// Animation
const clock = new THREE.Clock();
 
function animate() {
    requestAnimationFrame(animate);
 
    const delta = Math.min(clock.getDelta(), 0.05);
    const speed = 6 * delta;
 
    if (controls.isLocked) {
        if (keys.KeyW) controls.moveForward(speed);
        if (keys.KeyS) controls.moveForward(-speed);
        if (keys.KeyA) controls.moveRight(-speed);
        if (keys.KeyD) controls.moveRight(speed);
 
        if (keys.Space) camera.position.y += speed;
        if (keys.ShiftLeft) camera.position.y -= speed;
    }
 
    renderer.render(scene, camera);
}
 
animate();
 
// Fenstergröße
window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
 
    renderer.setSize(window.innerWidth, window.innerHeight);
});
 
</script>
</body>
</html>