# Teste-1
Studio ghibi
<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Personagens 3D Interativos</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {
  margin: 0;
  overflow: hidden;
  background: linear-gradient(#eaf6ff, #ffffff);
  font-family: Arial, sans-serif;
}

#ui {
  position: absolute;
  top: 15px;
  left: 15px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

button {
  padding: 10px 14px;
  border: none;
  border-radius: 10px;
  background: #5fa8d3;
  color: white;
  font-size: 14px;
  cursor: pointer;
}

button:hover {
  background: #4a94be;
}
</style>
</head>

<body>

<div id="ui">
  <button id="poseBtn">Trocar Pose</button>
  <button id="charBtn">Trocar Personagem</button>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158/examples/js/controls/OrbitControls.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.158/examples/js/loaders/GLTFLoader.js"></script>

<script>
let scene = new THREE.Scene();
scene.background = new THREE.Color(0xeaf6ff);

let camera = new THREE.PerspectiveCamera(
  60, window.innerWidth / window.innerHeight, 0.1, 100
);
camera.position.set(0, 2, 6);

let renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

let controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// 🌞 Luz
scene.add(new THREE.AmbientLight(0xffffff, 0.6));
let sun = new THREE.DirectionalLight(0xffffff, 1);
sun.position.set(5, 10, 5);
scene.add(sun);

// 📦 Loader
const loader = new THREE.GLTFLoader();

let characters = [];
let mixers = [];
let actions = [];
let activeChar = 0;
let currentPose = 0;

function loadCharacter(path, x) {
  loader.load(path, gltf => {
    let model = gltf.scene;
    model.position.set(x, 0, 0);
    scene.add(model);

    let mixer = new THREE.AnimationMixer(model);
    mixers.push(mixer);

    let charActions = gltf.animations.map(a => mixer.clipAction(a));
    charActions[0].play();

    characters.push(model);
    actions.push(charActions);

    if (characters.length > 1) model.visible = false;
  });
}

// 👤 Personagens
loadCharacter('char1.glb', 0);
loadCharacter('char2.glb', 0);

// 🔘 Trocar pose
document.getElementById("poseBtn").onclick = () => {
  currentPose = currentPose === 0 ? 1 : 0;

  actions[activeChar].forEach(a => a.stop());
  if (actions[activeChar][currentPose])
    actions[activeChar][currentPose].play();
};

// 🔄 Trocar personagem
document.getElementById("charBtn").onclick = () => {
  characters[activeChar].visible = false;
  activeChar = activeChar === 0 ? 1 : 0;
  characters[activeChar].visible = true;

  actions[activeChar].forEach(a => a.stop());
  actions[activeChar][0].play();
};

const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);
  let delta = clock.getDelta();
  mixers.forEach(m => m.update(delta));
  controls.update();
  renderer.render(scene, camera);
}
animate();

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>

</body>
</html>
