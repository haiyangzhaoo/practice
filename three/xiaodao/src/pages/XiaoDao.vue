<template>
    <div ref="container"></div>
</template>
<script setup>
import * as THREE from 'three'
// 控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// 水面
import { Water } from 'three/examples/jsm/objects/Water2'
// 导入gltf库
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader' // 解压
// 加载hdr图
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'
import {ref ,onMounted} from 'vue'

const container = ref(null)

// 场景
const scene = new THREE.Scene()

// 相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2000)
camera.position.set(-50, 50, 130)
camera.aspect = window.innerWidth / window.innerHeight
camera.updateProjectionMatrix()
scene.add(camera)

// 渲染
const renderer = new THREE.WebGLRenderer({
    // 是否执行抗锯齿
    antialias: true,
    // 是否使用对数深度缓存
    logarithmicDepthBuffer: true,
})
// 定义渲染器的输出编码
renderer.outputEncoding = THREE.sRGBEncoding
renderer.setSize(window.innerWidth, window.innerHeight)

// 控制器
const control = new OrbitControls(camera, renderer.domElement)
control.enableDamping = true

// 添加平面
// const planeGeometry = new THREE.PlaneGeometry(100, 100)
// const baseMesh = new THREE.MeshBasicMaterial({
//     color: 0xffffff
// })
// const cube = new THREE.Mesh(planeGeometry, baseMesh)
// scene.add(cube)

// 创建天空球体
const skyGeometry = new THREE.SphereGeometry(1000, 200, 200)
const skyMesh = new THREE.MeshBasicMaterial({
    map: new THREE.TextureLoader().load('./imgs/sky.jpeg')
})
skyGeometry.scale(1,1,-1)
const skyCube = new THREE.Mesh(skyGeometry, skyMesh)
scene.add(skyCube)

// 添加视频纹理
const video = document.createElement('video')
video.src = './imgs/sky.mp4'
video.loop = true

window.addEventListener('mousemove', () => {
    if (!video.paused) {
        video.play()
        skyMesh.map = new THREE.VideoTexture(video)
        skyMesh.map.needsUpdate = true
    }
})

// 创建水面
const waterGeometry = new THREE.CircleBufferGeometry(1000, 64)
const water = new Water(waterGeometry, {
    textureWidth: 1024,
    textureHeight: 1024,
    color: 0xeeeeff,
    flowDirection: new THREE.Vector2(1,1),
    scale:1,
})
// 水面放置水平
water.rotation.x = -Math.PI / 2
water.position.y = 3
scene.add(water)

// 加载小岛
const loader = new GLTFLoader()
const darcoLoader = new DRACOLoader()
darcoLoader.setDecoderPath('./draco/')
loader.setDRACOLoader(darcoLoader)
loader.load('./model/island2.glb', gltf => {
    console.log(111111,gltf )
    scene.add(gltf.scene)
})

// 加载环境纹理
const rgbLoader = new RGBELoader()
rgbLoader.loadAsync('./imgs/050.hdr').then(texture => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = texture
    scene.environment = texture
})

// 设置环境光
const light = new THREE.DirectionalLight(0xffffff, 1)
light.position.set(-100, 100, 10)
scene.add(light)

const render = () => {
    renderer.render(scene, camera)
    requestAnimationFrame(render)
}

onMounted(() => {
    render()
    
    container.value.appendChild(renderer.domElement)
    
    window.addEventListener('resize', () => {
        renderer.setSize(window.innerWidth, window.innerHeight)
        camera.aspect = window.innerWidth / window.innerHeight
        camera.updateProjectionMatrix()
    })
})
</script>