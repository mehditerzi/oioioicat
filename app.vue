<template>
  <div ref="arContainer" id="ar-container"></div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { ARButton } from 'three/examples/jsm/webxr/ARButton.js';

const arContainer = ref(null);

onMounted(() => {
  // Create the scene
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(70, window.innerWidth / window.innerHeight, 0.1, 1000);
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.xr.enabled = true;
  arContainer.value.appendChild(renderer.domElement);

  // Add AR button to the page
  document.body.appendChild(ARButton.createButton(renderer));

  // Add a light
  const light = new THREE.HemisphereLight(0xffffff, 0xbbbbff, 1);
  scene.add(light);

  // Load the .glb model
  const loader = new GLTFLoader();
  loader.load(
      '/path/to/your/model.glb', // Replace with your .glb model path
      (gltf) => {
        const model = gltf.scene;
        model.position.set(0, 0, -2); // Position the model
        model.scale.set(0.5, 0.5, 0.5); // Scale the model
        scene.add(model);

        // Animation logic
        const clock = new THREE.Clock();
        function animate() {
          renderer.setAnimationLoop(() => {
            const delta = clock.getDelta();
            if (model.rotation) {
              model.rotation.y += delta * 0.5; // Rotate the model for animation
            }
            renderer.render(scene, camera);
          });
        }
        animate();
      },
      undefined,
      (error) => {
        console.error('An error occurred while loading the model:', error);
      }
  );

  // Handle window resize
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
});
</script>

<style>
#ar-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
