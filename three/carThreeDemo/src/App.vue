<template>
  <div class="container" ref='container'></div>
  <div class="home">
    <h3>汽车展示与选配</h3>
    <div class="home-car-color">
      <h4>选择车身颜色</h4>
      <div class="home-car-color-container">
        <div v-for="(item, idx) in colors" v-bind:key="idx" v-on:click="selectColor(idx)">
          <div class="color-item" :style="{backgroundColor: item}"></div>
        </div>
      </div>
    </div>
    <div class="home-tiemo">
      <h4>选择贴膜材质</h4>
      <div class="home-tiemo-container">
        <div v-for="(item, idx) in materials" :key="idx" @click="selectMaterial(idx)">
          <div class="home-tiemo-item">{{item.name}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader' // 导入模型
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader' // 解压文件
import { ref, onMounted } from 'vue'

const container = ref(null)
let colors = ["red", "blue", "green", "gray", "orange", "purple"];
let materials = [
  { name: "磨砂", value: 1 },
  { name: "冰晶", value: 0 },
];
// 场景
const scene = new THREE.Scene()
// 相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(0,2,6)
camera.aspect = window.innerWidth / window.innerHeight
camera.updateProjectionMatrix()
scene.add(camera)
// 渲染器
const renderer = new THREE.WebGLRenderer(
   {
    // 抗锯齿
   antialias: true,
   }
)
renderer.setSize(window.innerWidth , window.innerHeight)

function render() {
  control.update()
  renderer.render(scene, camera)
  requestAnimationFrame(render)
  camera.updateProjectionMatrix()
  camera.aspect = window.innerWidth / window.innerHeight
}
// 控制器
const control = new OrbitControls(camera, renderer.domElement)
control.enableDamping = true
control.update()

window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth , window.innerHeight)
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
})


let wheels = [];
let carBody, frontCar, hoodCar, glassCar;
// 一堆材质
const bodyMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1, // 材质与金属的相似度
  roughness: 0.5, // 材质的粗糙程度
  clearcoat: 1, // 有些类似于车漆，碳纤，被水打湿的表面的材质需要在面上再增加一个透明的，具有一定反光特性的面。而且这个面说不定有一定的起伏与粗糙度。Clearcoat可以在不需要重新创建一个透明的面的情况下做到类似的效果。
  clearcoatRoughness: 0, // clear coat层的粗糙度
})

const frontMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
})

const hoodMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1,
  roughness: 0.5,
  clearcoat: 1,
  clearcoatRoughness: 0,
});
const wheelsMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xff0000,
  metalness: 1,
  roughness: 0.1,
});
const glassMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0,
  transmission: 1,
  transparent: true,
});

const selectColor = idx => {
  console.log(colors[idx])
  bodyMaterial.color.set(colors[idx]); // 两侧及后面
  frontMaterial.color.set(colors[idx]); // 进气栅栏
  hoodMaterial.color.set(colors[idx]); // 引擎盖
  wheelsMaterial.color.set(colors[idx]); // 轮圈
}

const selectMaterial = idx => {
  console.log(materials[idx])
  bodyMaterial.clearcoatRoughness = materials[idx].value
  frontMaterial.clearcoatRoughness = materials[idx].value
  hoodMaterial.clearcoatRoughness = materials[idx].value
}

onMounted(() => {
  container.value.appendChild(renderer.domElement)
  // 初始化渲染器背景
  renderer.setClearColor('#000')
  scene.background = new THREE.Color('#ccc')
  scene.environment = new THREE.Color('#ccc')
  render()

  // 加载网格地面
  const gridHelper = new THREE.GridHelper( 20, 20 );
  gridHelper.material.opacity = 0.2
  gridHelper.material.transparent = true
  scene.add(gridHelper);
  
  // 加载模型
  const loader = new GLTFLoader()
  const daroLoader = new DRACOLoader()
  daroLoader.setDecoderPath('./draco/gltf/')
  loader.setDRACOLoader(daroLoader)
  loader.load('./model/bmw01.glb', gltf => {
    const model = gltf.scene

    let childs = {
      Mesh: '尾灯粗线',
      Mesh_1: '尾灯细线',
      Box01001: '尾灯壳',
      Box02: '转向灯',
      Mesh014: '座椅皮',
      Mesh014_1: '座椅皮边',
      Box05: '座椅中间的护栏',
      Box06: '副驾靠背颜色'
    }

    model.traverse(child => {
      // if (child.isMesh) {
      //   console.log(child.name )
      //     if (child.name === 'Box06') {
      //       child.material = new THREE.MeshStandardMaterial({
      //         color: 'green' 
      //       })
      //     }
      // }
      // 判断是否是轮毂
      if (child.name.includes('轮毂')) {
        child.material = wheelsMaterial
        wheels.push(child)
      }
      // 判断是否是车身
      if (child.name.includes('Mesh002')) {
        child.material = bodyMaterial
        carBody = child
      }
      // 判断是否是前脸
      if (child.isMesh && child.name.includes("前脸")) {
        child.material = frontMaterial;
        frontCar = child;
      }
      // 判断是否是引擎盖
      if (child.isMesh && child.name.includes("引擎盖_1")) {
        child.material = hoodMaterial;
        hoodCar = child;
      }
      // 判断是否是挡风玻璃
      if (child.isMesh && child.name.includes("挡风玻璃")) {
        child.material = glassMaterial;
        glassCar = child;
      }
    })

    scene.add(model)
  })

  // 灯光照射
  const light1 = new THREE.DirectionalLight(0xffffff, 1);
  light1.position.set(0, 0, 10);
  scene.add(light1);
  const light2 = new THREE.DirectionalLight(0xffffff, 1);
  light2.position.set(0, 0, -10);
  scene.add(light2);
  const light3 = new THREE.DirectionalLight(0xffffff, 1);
  light3.position.set(10, 0, 0);
  scene.add(light3);
  const light4 = new THREE.DirectionalLight(0xffffff, 1);
  light4.position.set(-10, 0, 0);
  scene.add(light4);
  const light5 = new THREE.DirectionalLight(0xffffff, 1);
  light5.position.set(0, 10, 0);
  scene.add(light5);
  const light6 = new THREE.DirectionalLight(0xffffff, 0.3);
  light6.position.set(5, 10, 0);
  scene.add(light6);
  const light7 = new THREE.DirectionalLight(0xffffff, 0.3);
  light7.position.set(0, 10, 5);
  scene.add(light7);
  const light8 = new THREE.DirectionalLight(0xffffff, 0.3);
  light8.position.set(0, 10, -5);
  scene.add(light8);
  const light9 = new THREE.DirectionalLight(0xffffff, 0.3);
  light9.position.set(-5, 10, 0);
  scene.add(light9);

})
</script>


<style scoped>
.home {
  position: fixed;
  top: 0;
  right: 20px;
}
.home-car-color-container, .home-tiemo-container {
  display: flex;
  align-items: center;
  cursor: pointer;
}
.home-car-color-container div {
  width: 50px;
  height: 50px;
  border-radius: 5px;
  margin-left: 5px;
}
.home-tiemo-container div {
  margin-left: 5px;
}
</style>
