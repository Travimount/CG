<template>
  <div ref="container" class="full-screen-container"></div>
</template>

<script>

import * as THREE from 'three';
import Stats from 'three/addons/libs/stats.module.js';
import { GUI } from 'three/addons/libs/lil-gui.module.min.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import '../Shader/MeshShader'
import {MeshShader} from "@/Shader/MeshShader";
export default {
  name: 'HelloWorld',
  data() {
    return {
      bulbLuminousPowers: {
        '110000 lm ': 110000,
        '3500 lm ': 3500,
        '1700 lm ': 1700,
        '800 lm ': 800,
        '400 lm ': 400,
        '180 lm ': 180,
        '20 lm ': 20,
        'Off': 0
      },
      hemiLuminousIrradiances: {
        '0.0001 lx ': 0.0001,
        '0.002 lx ': 0.002,
        '0.5 lx ': 0.5,
        '3.4 lx ': 3.4,
        '50 lx ': 50,
        '100 lx ': 100,
        '350 lx ': 350,
        '400 lx ': 400,
        '1000 lx ': 1000,
        '18000 lx ': 18000,
        '50000 lx ': 50000
      },
      // 参数设置
      params: {
        shadows: true,
        exposure: 0.68,
        bulbPower: '800 lm ',
        hemiIrradiance: '0.0001 lx '
      }
    };
  },
  mounted() {
    this.initThreeJS();
    this.animate();
  },

  methods: {
    initThreeJS() {
      const container = this.$refs.container;

      // Stats
      this.stats = new Stats();
      container.appendChild(this.stats.dom);

      // 设置相机
      this.camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 100);
      this.camera.position.set(-4, 2, 4);

      // 设置场景
      this.scene = new THREE.Scene();

      // 设置灯光、材质、地板、球体、盒子等对象
      const bulbGeometry = new THREE.SphereGeometry(0.02, 16, 8);
      this.bulbLight = new THREE.PointLight(0xffee88, 1, 100, 2);
      this.bulbMat = new THREE.MeshStandardMaterial({
        emissive: 0xffffee,
        emissiveIntensity: 1,
        color: 0x000000
      });
      this.bulbLight.add(new THREE.Mesh(bulbGeometry, this.bulbMat));
      this.bulbLight.position.set(0, 2, 0);
      this.bulbLight.castShadow = true;
      this.scene.add(this.bulbLight);

      // 半球光源
      this.hemiLight = new THREE.HemisphereLight(0xddeeff, 0x0f0e0d, 0.02);
      this.scene.add(this.hemiLight);

      //材质
      this.initMaterials();

      //地板
      const floorGeometry = new THREE.PlaneGeometry(15, 15);
      const floorMesh = new THREE.Mesh(floorGeometry, this.floorMat);
      floorMesh.receiveShadow = true;
      floorMesh.rotation.x = -Math.PI / 2.0;
      this.scene.add(floorMesh);

      //球体
      const ballGeometry = new THREE.SphereGeometry(0.5, 32, 32);
      this.ballMesh = new THREE.Mesh(ballGeometry, this.ballMat);
      this.ballMesh.position.set(0, 1, 3);
      this.ballMesh.rotation.y = Math.PI;
      this.ballMesh.castShadow = true;
      this.scene.add(this.ballMesh);

      //盒子
      const boxGeometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);
      this.createBoxMesh(boxGeometry, -3, 0.25, 0);
      this.createBoxMesh(boxGeometry, 0, 0.25, -3);
      this.createBoxMesh(boxGeometry, 3, 0.25, 0);

      //渲染器
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.shadowMap.enabled = true;
      this.renderer.toneMapping = THREE.ReinhardToneMapping;
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      container.appendChild(this.renderer.domElement);

      // 控制器
      const controls = new OrbitControls(this.camera, this.renderer.domElement);
      controls.minDistance = 1;
      controls.maxDistance = 20;

      //窗口尺寸变化
      window.addEventListener('resize', this.onWindowResize);

      // GUI
      const gui = new GUI();
      gui.add(this.params, 'hemiIrradiance', Object.keys(this.hemiLuminousIrradiances));
      gui.add(this.params, 'bulbPower', Object.keys(this.bulbLuminousPowers));
      gui.add(this.params, 'exposure', 0, 1);
      gui.add(this.params, 'shadows');
      gui.open();
    },

    animate() {
      //动画循环
      requestAnimationFrame(this.animate);
      this.render();
    },
    //渲染函数
    render() {
      this.renderer.toneMappingExposure = Math.pow(this.params.exposure, 5.0); // Bright scenes
      this.renderer.shadowMap.enabled = this.params.shadows;
      this.bulbLight.castShadow = this.params.shadows;

      if (this.params.shadows !== this.previousShadowMap) {
        this.ballMat.needsUpdate = true;
        this.cubeMat.needsUpdate = true;
        this.floorMat.needsUpdate = true;
        this.previousShadowMap = this.params.shadows;
      }

      this.bulbLight.power = this.bulbLuminousPowers[this.params.bulbPower];
      this.bulbMat.emissiveIntensity = this.bulbLight.intensity / Math.pow(0.02, 2.0);

      this.hemiLight.intensity = this.hemiLuminousIrradiances[this.params.hemiIrradiance];
      const time = Date.now() * 0.0005;
      const radius = 1.5;
      this.bulbLight.position.x = Math.cos(time) * radius;
      this.bulbLight.position.z = Math.sin(time) * radius;
      this.bulbLight.position.y = 1.5;


      const rotationSpeed = 0.005; // 设置球体的旋转速度
      this.ballMesh.rotation.y += rotationSpeed; // 更新球体的旋转

      this.renderer.render(this.scene, this.camera);
      this.stats.update();
    },

    onWindowResize() {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    },

    initMaterials() {
      this.textureLoader = new THREE.TextureLoader();
      // Floor Material
      this.floorMat = new MeshShader({
        roughness: 0.8,
        color: 0xffffff,
        metalness: 0.2,
        bumpScale: 1
      });
      this.textureLoader.load('img/hardwood2_diffuse.jpg', (map) => {
        map.wrapS = THREE.RepeatWrapping;
        map.wrapT = THREE.RepeatWrapping;
        map.anisotropy = 4;
        map.repeat.set(10, 24);
        map.colorSpace = THREE.SRGBColorSpace;
        this.floorMat.map = map;
        this.floorMat.needsUpdate = true;
      });
      this.textureLoader.load('img/hardwood2_bump.jpg', (map) => {
        map.wrapS = THREE.RepeatWrapping;
        map.wrapT = THREE.RepeatWrapping;
        map.anisotropy = 4;
        map.repeat.set(10, 24);
        this.floorMat.bumpMap = map;
        this.floorMat.needsUpdate = true;
      });
      this.textureLoader.load('img/hardwood2_roughness.jpg', (map) => {
        map.wrapS = THREE.RepeatWrapping;
        map.wrapT = THREE.RepeatWrapping;
        map.anisotropy = 4;
        map.repeat.set(10, 24);
        this.floorMat.roughnessMap = map;
        this.floorMat.needsUpdate = true;
      });

      // Cube Material
      this.cubeMat = new MeshShader({
        roughness: 0.7,
        color: 0xffffff,
        bumpScale: 1,
        metalness: 0.2
      });
      this.textureLoader.load('img/brick_diffuse.jpg', (map) => {
        map.wrapS = THREE.RepeatWrapping;
        map.wrapT = THREE.RepeatWrapping;
        map.anisotropy = 4;
        map.repeat.set(1, 1);
        map.colorSpace = THREE.SRGBColorSpace;
        this.cubeMat.map = map;
        this.cubeMat.needsUpdate = true;
      });
      this.textureLoader.load('img/brick_bump.jpg', (map) => {
        map.wrapS = THREE.RepeatWrapping;
        map.wrapT = THREE.RepeatWrapping;
        map.anisotropy = 4;
        map.repeat.set(1, 1);
        this.cubeMat.bumpMap = map;
        this.cubeMat.needsUpdate = true;
      });

      // Ball Material
      this.ballMat = new MeshShader({
        color: 0xffffff,
        roughness: 0.5,
        metalness: 1.0
      });
      this.textureLoader.load('img/earth_atmos_2048.jpg', (map) => {
        map.anisotropy = 4;
        map.colorSpace = THREE.SRGBColorSpace;
        this.ballMat.map = map;
        this.ballMat.needsUpdate = true;
      });
      this.textureLoader.load('img/earth_specular_2048.jpg', (map) => {
        map.anisotropy = 4;
        map.colorSpace = THREE.SRGBColorSpace;
        this.ballMat.metalnessMap = map;
        this.ballMat.needsUpdate = true;
      });
    },

    //创建盒子网格
    createBoxMesh(geometry, x, y, z) {
      const mesh = new THREE.Mesh(geometry, this.cubeMat);
      mesh.position.set(x, y, z);
      mesh.castShadow = true;
      this.scene.add(mesh);
    },
  }
};
</script>

<style>
.full-screen-container {
  width: 100vw;   /* 视窗的全宽 */
  height: 100vh;  /* 视窗的全高 */
  margin: 0;
  padding: 0;
  overflow: hidden;
}

</style>