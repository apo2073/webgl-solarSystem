<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const canvasRef = ref(null);
const speedFactor = ref(1);

onMounted(() => {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 5000);
  camera.position.set(0, 500, 1500);

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  canvasRef.value.appendChild(renderer.domElement);

  const light = new THREE.PointLight(0xffffff, 3, 5000);
  light.position.set(0, 0, 0);
  scene.add(light, new THREE.AmbientLight(0x606060));

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.zoomSpeed = 0.3;

  // 🌞 태양
  const sunGeometry = new THREE.SphereGeometry(50, 32, 32);
  const sunMaterial = new THREE.MeshBasicMaterial({ color: 0xffcc00, emissive: 0xffaa00 });
  const sun = new THREE.Mesh(sunGeometry, sunMaterial);
  scene.add(sun);

  // 🌍 행성 데이터 
  const AU = 200;
  const planets = [
    { name: "Mercury", size: 4, orbitRadius: 0.39 * AU, eccentricity: 0.205, color: 0xa8a8a8, speed: 0.24 },
    { name: "Venus", size: 9, orbitRadius: 0.72 * AU, eccentricity: 0.007, color: 0xe8ac46, speed: 0.615 },
    { name: "Earth", size: 10, orbitRadius: 1.0 * AU, eccentricity: 0.016, color: 0x9be394, speed: 1.0 },
    { name: "Mars", size: 5, orbitRadius: 1.52 * AU, eccentricity: 0.093, color: 0xf58664, speed: 1.88 },
    { name: "Jupiter", size: 50, orbitRadius: 5.2 * AU, eccentricity: 0.048, color: 0x91520a, speed: 11.86 },
    { name: "Saturn", size: 45, orbitRadius: 9.58 * AU, eccentricity: 0.054, color: 0x855c2d, speed: 29.46 },
    { name: "Uranus", size: 20, orbitRadius: 19.2 * AU, eccentricity: 0.047, color: 0x45a0f5, speed: 84.01 },
    { name: "Neptune", size: 19, orbitRadius: 30.1 * AU, eccentricity: 0.008, color: 0x69def0, speed: 164.8 }
  ];

  const planetMeshes = [];

  planets.forEach(planet => {
    // 🪐 행성 생성
    const geometry = new THREE.SphereGeometry(planet.size, 32, 32);
    const material = new THREE.MeshStandardMaterial({ color: planet.color, emissive: planet.color, emissiveIntensity: 0.5 });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);
    planetMeshes.push({ mesh, ...planet, angle: Math.random() * Math.PI * 2 });

    // 🌀 타원 궤도
    const a = planet.orbitRadius;
    const b = a * Math.sqrt(1 - planet.eccentricity ** 2);
    const focus = a * planet.eccentricity;

    const curve = new THREE.EllipseCurve(-focus, 0, a, b);
    const points = curve.getPoints(100);
    const orbitGeometry = new THREE.BufferGeometry().setFromPoints(points.map(p => new THREE.Vector3(p.x, 0, p.y)));
    const orbitMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
    const orbitLine = new THREE.Line(orbitGeometry, orbitMaterial);
    scene.add(orbitLine);
  });

  function animate() {
    requestAnimationFrame(animate);

    planetMeshes.forEach(planet => {
      planet.angle += (0.002 * speedFactor.value / planet.speed);
      const a = planet.orbitRadius;
      const b = a * Math.sqrt(1 - planet.eccentricity ** 2);
      const focus = a * planet.eccentricity;

      planet.mesh.position.x = (a * Math.cos(planet.angle)) - focus;
      planet.mesh.position.z = b * Math.sin(planet.angle);
      planet.mesh.rotation.y += 0.02;
    });

    controls.update();
    renderer.render(scene, camera);
  }

  animate();

  window.addEventListener("resize", () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
  });
});
</script>

<template>
  <div ref="canvasRef" class="webgl-container"></div>
  <input type="range" min="0.1" max="200" step="0.1" v-model="speedFactor" class="speed-slider" />
</template>

<style>
.webgl-container {
  width: 100vw;
  height: 100vh;
}
.speed-slider {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}
</style>
