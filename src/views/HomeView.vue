<template>
  <div></div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader'
import { RGBELoader } from 'three/examples/jsm/loaders/RGBELoader'
import { Water } from 'three/examples/jsm/objects/Water2'

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
// 设置渲染器色调映射
renderer.outputEncoding = THREE.sRGBEncoding // 设置色调输出颜色编码
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 0.5 // 设置渲染器色调映射的曝光程度
// 设置渲染器开启阴影的计算
renderer.shadowMap.enabled = true
// 设置水面纹理效果

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement)
// controls.enableDamping = true

// 初始化loader
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('./draco/')
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)

// 加载环境纹理
let rgbeLoader = new RGBELoader()
rgbeLoader.load('./textures/sky.hdr', (texture) => {
  // 设置纹理映射，球面
  texture.mapping = THREE.EquirectangularReflectionMapping
  scene.background = texture
  scene.environment = texture
})

// 加载模型
gltfLoader.load('./model/scene.glb', (gltf) => {
  const model = gltf.scene
  model.traverse((child) => {
    if (child.name === 'Plane') {
      child.visible = false
    }
    if (child.isMesh) {
      child.castShadow = true //设置物体投射阴景
      child.receiveShadow = true //设置物体接收阴影
    }
  })
  scene.add(model)
})
// 创建水面
const waterGeometry = new THREE.CircleGeometry(300, 32)
const water = new Water(waterGeometry, {
  textureWidth: 1024,
  textureHeight: 1024,
  color: 0xeeeeff,
  flowDirection: new THREE.Vector2(1, 1),
  scale: 100
})
water.rotation.x = -Math.PI / 2
water.position.y = -0.4
scene.add(water)

// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 1)
light.position.set(0, 50, 0)
scene.add(light)

// 添加点光源（房子里加个灯泡）
const pointLight = new THREE.PointLight(0xffffff, 10)
pointLight.position.set(0.1, 2.4, 0)
pointLight.castShadow = true
scene.add(pointLight)

// 创建点光源组
const pointLightGroup = new THREE.Group()
pointLightGroup.position.set(-8, 2.5, -1.5)
let radius = 3
let pointLightArr = []
for (let i = 0; i < 3; i++) {
  // 创建球体当灯泡
  const sphereGeometry = new THREE.SphereGeometry(0.2, 32, 32)
  const sphereMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff })
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
  sphere.position.set(
    radius * Math.cos((i * 2 * Math.PI) / 3),
    Math.cos((i * 2 * Math.PI) / 3),
    radius * Math.sin((i * 2 * Math.PI) / 3)
  )

  let pointLight = new THREE.PointLight(0xffffff, 1)
  sphere.add(pointLight)

  pointLightGroup.add(sphere)
}
scene.add(pointLightGroup)

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
