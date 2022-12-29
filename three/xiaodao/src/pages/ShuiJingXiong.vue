<template>
    <div ref="container"></div>
</template>
<script setup>
import * as THREE from 'three'
// 控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// 导入gltf库
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
// 加载hdr图
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

const render = () => {
    // 更新控制器
    control.update()
    renderer.render(scene, camera)
    requestAnimationFrame(render)
}

// 加载环境
const texture = new THREE.TextureLoader().load('./imgs/sky.jpeg')
texture.mapping = THREE.EquirectangularRefractionMapping
scene.background = texture
scene.environment = texture

// 加载模型
const gltfLoader = new GLTFLoader()
const loader = gltfLoader.load('./imgs/熊.gltf', gltf => {
    const model = gltf.scene.children[0]

    // 修改材质
    model.material = new THREE.MeshPhongMaterial({
        color: 0xffffff,
        envMap: texture,
        refractionRatio: 0.7,
        reflectivity: 0.99
    })

    model.scale.set(20,20,20)
    scene.add(model)
})

// 添加环境光
const allight = new THREE.AmbientLight(0xffffff, 1)
scene.add(allight)

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