<template>
  <div ref="sceneContainer"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

export default {
  mounted() {
    this.initThreeJS();
  },
  methods: {
    initThreeJS() {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio); // 提高分辨率
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setClearColor(0x666666); // 设置背景颜色为中灰色
      this.$refs.sceneContainer.appendChild(renderer.domElement);

      // 设置初始相机位置，确保模型从正面被观看
      camera.position.set(0, 0, 30); // 设置一个较远的初始位置，Y轴稍微高一点，Z轴较远一点
      camera.lookAt(0, 0, 0); // 设置相机看向模型的中心

      // 添加光源
      const light1 = new THREE.DirectionalLight(0xffffff, 1);
      light1.position.set(5, 5, 5).normalize();
      scene.add(light1);

      const light2 = new THREE.DirectionalLight(0xffffff, 0.5);
      light2.position.set(-5, -5, 5);
      scene.add(light2);

      const hemisphereLight = new THREE.HemisphereLight(0xffffbb, 0x080820, 1);
      scene.add(hemisphereLight);

      const ambientLight = new THREE.AmbientLight(0x404040, 2); // 增强环境光的强度
      scene.add(ambientLight);

      // 加载模型
      const loader = new GLTFLoader();
      loader.load(
        '/buddha_-_four_faces.glb',
        (gltf) => {
          const model = gltf.scene;
          model.position.set(0, 0, 0);
          model.scale.set(1, 1, 1);
          scene.add(model);
          this.animate(renderer, scene, camera, controls);
        },
        (xhr) => {
          console.log((xhr.loaded / xhr.total * 100) + '% loaded');
        },
        (error) => {
          console.error('An error happened', error);
        }
      );

      // 设置 OrbitControls
      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.01; // 增强阻尼感
      controls.enableZoom = true;

      // 控制旋转轴
      controls.enableRotate = true;
      controls.mouseButtons = {
        LEFT: THREE.MOUSE.ROTATE,
        MIDDLE: THREE.MOUSE.DOLLY,
        RIGHT: THREE.MOUSE.NONE,
      };

      // 禁用 panning （平移）功能
      controls.enablePan = false;

      // 设置缩放的边界，避免穿透模型
      controls.minDistance = 15; // 最小缩放距离，确保相机不会太接近模型
      controls.maxDistance = 30; // 最大缩放距离，防止缩放太远

      // 渲染循环
      this.animate(renderer, scene, camera, controls);
    },
    animate(renderer, scene, camera, controls) {
      requestAnimationFrame(() => this.animate(renderer, scene, camera, controls));
      controls.update(); // 每帧更新控制器
      renderer.render(scene, camera);
    },
  },
};
</script>

<style scoped>
/* 确保容器充满整个窗口 */
div {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
