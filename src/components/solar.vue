<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

console.log('init')
const canvasRef = ref(null);
const speedFactor = ref(1.0);

const AU = 15;

const planets = [
  { name: "Mercury", size: 0.5, semiMajorAxis: 0.39 * AU, eccentricity: 0.205, color: 0xa8a8a8, period: 0.24 },
  { name: "Venus", size: 0.8, semiMajorAxis: 0.72 * AU, eccentricity: 0.007, color: 0xe8ac46, period: 0.615 },
  { name: "Earth", size: 1, semiMajorAxis: 1.0 * AU, eccentricity: 0.016, color: 0x9be394, period: 1.0 },
  { name: "Mars", size: 0.7, semiMajorAxis: 1.52 * AU, eccentricity: 0.093, color: 0xf58664, period: 1.88 },
  { name: "Jupiter", size: 3, semiMajorAxis: 5.2 * AU, eccentricity: 0.048, color: 0x91520a, period: 11.86 },
  { name: "Saturn", size: 2.5, semiMajorAxis: 9.58 * AU, eccentricity: 0.054, color: 0x855c2d, period: 29.46 },
  { name: "Uranus", size: 1.8, semiMajorAxis: 19.2 * AU, eccentricity: 0.047, color: 0x45a0f5, period: 84.01 },
  { name: "Neptune", size: 1.7, semiMajorAxis: 30.1 * AU, eccentricity: 0.008, color: 0x69def0, period: 164.8 }
];

onMounted(() => {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.set(0, 100, 250);
  console.log('scene set')

  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  canvasRef.value.appendChild(renderer.domElement);
  console.log('renderer set')

  const light = new THREE.PointLight(0xffffff, 3, 500);
  light.position.set(0, 50, 0);
  scene.add(light, new THREE.AmbientLight(0x606060));

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.zoomSpeed = 0.3;
  console.log('controls set')

  const sunGeometry = new THREE.SphereGeometry(3.5, 32, 32);
  const sunMaterial = new THREE.MeshBasicMaterial({ color: 0xffcc00 });
  const sun = new THREE.Mesh(sunGeometry, sunMaterial);
  scene.add(sun);
  console.log('star "sun" added to scene')

  const planetMeshes = [];

  function createLabel(text) {
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");
    canvas.width = 128;
    canvas.height = 32;
    context.font = "20px Arial";
    context.fillStyle = "white";
    context.fillText(text, 0, 20);

    const texture = new THREE.CanvasTexture(canvas);
    const spriteMaterial = new THREE.SpriteMaterial({ map: texture });
    const sprite = new THREE.Sprite(spriteMaterial);
    sprite.scale.set(10, 2.5, 1);
    return sprite;
  }

  const EARTH_ORBIT_TIME = 10;
  const EARTH_ANGULAR_SPEED = (2 * Math.PI) / EARTH_ORBIT_TIME; 

  planets.forEach(planet => {
    const geometry = new THREE.SphereGeometry(planet.size, 32, 32);
    const material = new THREE.MeshStandardMaterial({ color: planet.color, emissive: planet.color, emissiveIntensity: 0.5 });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    const a = planet.semiMajorAxis;
    const b = a * Math.sqrt(1 - planet.eccentricity ** 2);
    const focus = a * planet.eccentricity;
    const angularSpeed = EARTH_ANGULAR_SPEED / planet.period;

    const label = createLabel(planet.name);
    scene.add(label);

    const curve = new THREE.EllipseCurve(0, 0, a, b, 0, 2 * Math.PI, false);
    const points = curve.getPoints(100);
    const orbitGeometry = new THREE.BufferGeometry().setFromPoints(points.map(p => new THREE.Vector3(p.x - focus, 0, p.y)));
    const orbitMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
    const orbitLine = new THREE.Line(orbitGeometry, orbitMaterial);
    scene.add(orbitLine);

    planetMeshes.push({ name: planet.name, mesh, label, curve, a, b, eccentricity: planet.eccentricity, focus, angularSpeed, angle: Math.random() * Math.PI * 2 });
  });

  const lodGroup = new THREE.Group();
  planetMeshes.forEach(planet => {
    const lod = new THREE.LOD();
    lod.addLevel(planet.mesh, planet.a * 0.5);
    lodGroup.add(lod);
  });
  scene.add(lodGroup);

  planetMeshes.forEach(p => {
    console.log(`planet ${p.name} is in planetMeshes`)
    console.log(`planet ${p.name}'s angular speed is ${p.angularSpeed}`)
  });

  let lastTime = 0;
  function animate(time) {
    requestAnimationFrame(animate);

    const deltaTime = (time - lastTime) / 1000; 
    lastTime = time;

    planetMeshes.forEach(planet => {
      planet.angle += planet.angularSpeed * speedFactor.value * deltaTime;
      if (planet.angle >= 2 * Math.PI) planet.angle -= 2 * Math.PI;

      const t = planet.angle / (2 * Math.PI);
      const point = planet.curve.getPoint(t);
      const x = point.x - planet.focus;
      const z = point.y;

      planet.mesh.position.set(x, 0, z);
      planet.mesh.rotation.y += 0.02;

      planet.label.position.set(x, planet.mesh.geometry.parameters.radius + 2, z);
      planet.label.lookAt(camera.position);
    });

    controls.update();
    renderer.render(scene, camera);
  }

  animate(0);

  window.addEventListener("resize", () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = (window.innerWidth / window.innerHeight);
    camera.updateProjectionMatrix();
  });
})
</script>

<template>
  <div ref="canvasRef" class="webgl-container"></div>
  <input type="range" min="1" max="100" step="1" v-model="speedFactor" class="speed-slider" />
</template>

<style>
.webgl-container {
  width: 100vw;
  height: 100vh;
  background: #000;
}
.speed-slider {
  position: absolute;
  top: 30px;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
}
</style>