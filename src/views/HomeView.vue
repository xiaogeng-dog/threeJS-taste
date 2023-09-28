<template>
  <div></div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader'
import gsap from 'gsap'
const COLLISION_SCENE_URL = new URL('../assets/models/scene_collision.glb', import.meta.url).href
const STATIC_SCENE_URL = new URL('../assets/models/scene_desk_obj.glb', import.meta.url).href

// 初始化场景
const scene = new THREE.Scene()
// 初始化相机
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
camera.position.set(-3.23, 2.98, 4.06)
// camera.aspect = window.innerWidth / window.innerHeight
// 更新投影矩阵
camera.updateProjectionMatrix()
// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  // 设置抗锯齿
  antialias: true
})
renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement)

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true

// 初始化loader
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('./draco/')
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)

// 加载模型
gltfLoader.load('./model/scene.glb', (gltf) => {
  scene.add(gltf.scene)
})

// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 1)
light.position.set(0, 50, 0)
scene.add(light)

function render() {
  requestAnimationFrame(render)
  renderer.render(scene, camera)
  controls.update()
}
render()
</script>

<style lang="less" scoped>
canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
