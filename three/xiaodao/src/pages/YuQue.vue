<template>
    <div class="container" ref="container"></div>
</template>
<script setup>
    import { ref, onMounted, } from 'vue'
    import * as THREE from 'three'
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

    const container = ref(null)
    
    // 场景
    const scene = new THREE.Scene()
    
    // 相机
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
    camera.position.set(0,3,3)
    scene.add(camera)

    // 物体
    const boxMetrail = new THREE.BoxGeometry(1,1,1)
    const mesh = new THREE.MeshBasicMaterial({
        color: 0xff0000
    })
    const cube = new THREE.Mesh(boxMetrail, mesh)
    scene.add(cube)

    // 渲染
    const renderer = new THREE.WebGLRenderer()
    renderer.setSize(window.innerWidth, window.innerHeight)
    
    // 控制器
    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    // 辅助线
    const axesHelper = new THREE.AxesHelper(5)
    scene.add(axesHelper)

    const render = () => {
        renderer.render(scene, camera)

        requestAnimationFrame(render)
    }

    // 监听窗口大小改变
    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight
        camera.updateProjectionMatrix()
        renderer.setSize(window.innerWidth, window.innerHeight)
    })

    onMounted(() => {
        container.value.appendChild(renderer.domElement)

        render()
    })

</script>
