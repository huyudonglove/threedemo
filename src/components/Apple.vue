<script setup>
import { onMounted, ref } from 'vue'
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { useMousePosition } from '../hooks/useMousePosition.js';
const scene = new THREE.Scene();
const { x, y } = useMousePosition();

const camera = new THREE.PerspectiveCamera(
  75, window.innerWidth / window.innerHeight, 0.1, 1000
);
camera.position.set(2, 2, 5);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);


// 2. 添加控制器
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// 3. 添加光源
const light = new THREE.DirectionalLight(0xffffff, 10);
light.position.set(50, 10, 10);
scene.add(light);

const ambient = new THREE.AmbientLight(0xffffff, 1);
scene.add(ambient);


const loader = new GLTFLoader();
loader.load(
  '/public/apple0613/apple.gltf', // ✅ 替换为你自己的路径
  (gltf) => {
        let baseMesh = null;
        gltf.scene.traverse((child) => {
            if (child.isMesh && !baseMesh) baseMesh = child;
        });

        if (baseMesh) {
            const geometry = baseMesh.geometry;
            const material = baseMesh.material;
            const instanced = new THREE.InstancedMesh(geometry, material, 100);

            const dummy = new THREE.Object3D();
            for (let i = 0; i < 100; i++) {
                dummy.position.set(Math.random() * 10 - 5, 0, Math.random() * 10 - 5);
                dummy.scale.set(10,10,10)
                dummy.updateMatrix();
                instanced.setMatrixAt(i, dummy.matrix);
            }

            scene.add(instanced);
        }
  },
  (xhr) => {
    console.log(`模型加载中：${(xhr.loaded / xhr.total * 100).toFixed(2)}%`);
  },
  (error) => {
    console.error('模型加载失败:', error);
  }
);
renderer.setAnimationLoop(() => {
  controls.update(); // 必须更新控制器
  renderer.render(scene, camera);
  light.position.set(x.value/100,10,5)
});

const webglCon = ref(null);
onMounted(() => {
    //console.log(webglCon);
    webglCon.value.appendChild(renderer.domElement);
})

</script>
<template>
    
     <div ref="webglCon" class="webglCon"></div>
</template>
<style scoped>
.webglCon {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}

</style>