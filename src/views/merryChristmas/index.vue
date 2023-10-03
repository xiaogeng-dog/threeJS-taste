<template>
  <div
    class="scens"
    style="position: fixed; top: 0; left: 0; z-index: 10; pointer-events: none; transition: all 1s"
    :style="{ transform: `translate3d(0,${-index * 100}vh,0)` }"
  >
    <div v-for="(item, index) in scenes" :key="index" style="width: 100vw; height: 100vh">
      <h1 style="padding: 100px 50px; font-size: 50px; color: #fff">{{ item.text }}</h1>
    </div>
  </div>
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
// 设置物理渲染效果
renderer.physicallyCorrectLights = true
// 设置水面纹理效果

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement)
controls.target.set(-8, 2, 0)
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
const pointLight = new THREE.PointLight(0xffffff, 50)
pointLight.position.set(0.1, 2.0, 0)
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
  const sphereMaterial = new THREE.MeshStandardMaterial({
    color: 0xffffff,
    emissive: 0xffffff,
    emissiveIntensity: 10
  })
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
  sphere.position.set(
    radius * Math.cos((i * 2 * Math.PI) / 3),
    Math.cos((i * 2 * Math.PI) / 3),
    radius * Math.sin((i * 2 * Math.PI) / 3)
  )

  let pointLight = new THREE.PointLight(0xffffff, 50)
  sphere.add(pointLight)
  pointLightArr.push(sphere)
  pointLightGroup.add(sphere)
}
scene.add(pointLightGroup)

// 使用补间函数，从0到2Π，使灯泡旋转
let options = {
  angle: 0
}
gsap.to(options, {
  angle: Math.PI * 2,
  duration: 10,
  repeat: 1,
  ease: 'linear',
  onUpdate: () => {
    pointLightGroup.rotation.y = options.angle
    pointLightArr.forEach((item, index) => {
      item.position.set(
        radius * Math.cos((index * 2 * Math.PI) / 3),
        Math.cos((index * 2 * Math.PI) / 3 + options.angle * 5),
        radius * Math.sin((index * 2 * Math.PI) / 3)
      )
    })
  }
})

function render() {
  requestAnimationFrame(render)
  renderer.render(scene, camera)
  controls.update()
}
render()
// 使用补间动画移动相机
let timeLine1 = gsap.timeline()
let timeLine2 = gsap.timeline()

// 定义相机的移动函数
function translateCamera(position, target) {
  timeLine1.to(camera.position, {
    x: position.x,
    y: position.y,
    z: position.z,
    duration: 1,
    ease: 'power2.inout'
  })

  timeLine2.to(controls.target, {
    x: target.x,
    y: target.y,
    z: target.z,
    duration: 1,
    ease: 'power2.inout'
  })
}

// 切换相机移动位置函数
let scenes = [
  {
    text: '圣诞快乐',
    callBack: () => {
      // 执行函数切换位置
      translateCamera(new THREE.Vector3(-3.23, 3, 4.06), new THREE.Vector3(-8, 2, 0))
    }
  },
  {
    text: '感谢在这么大的世界遇见了你',
    callBack: () => {
      // 执行函数切换位置
      translateCamera(new THREE.Vector3(0, 0, 23), new THREE.Vector3(0, 0, 0))
    }
  },
  {
    text: '愿与你探寻世界的每一个角落',
    callBack: () => {
      // 执行函数切换位置
      translateCamera(new THREE.Vector3(10, 3, 0), new THREE.Vector3(5, 2, 0))
    }
  },
  {
    text: '愿将天上的星星摘下送给你',
    callBack: () => {
      // 执行函数切换位置
      translateCamera(new THREE.Vector3(0, 0, 23), new THREE.Vector3(0, 0, 0))
      makeHeart()
    }
  },
  {
    text: '愿你一生健康快乐',
    callBack: () => {
      // 执行函数切换位置
      translateCamera(new THREE.Vector3(-20, 1.3, 6.6), new THREE.Vector3(0, 0, 0))
    }
  }
]
let index = ref(0)
let isAnimate = false
window.addEventListener(
  'wheel',
  (e) => {
    if (isAnimate) return
    isAnimate = true
    if (e.deltaY > 0) {
      index.value++
      if (index.value > scenes.length - 1) {
        index.value = 0
        restoreHeart()
      }
      scenes[index.value].callBack()
      setTimeout(() => {
        isAnimate = false
      }, 1000)
    }
  },
  false
)

// 实例化创建漫天星星
let starsInstance = new THREE.InstancedMesh(
  new THREE.SphereGeometry(0.1, 32, 32),
  new THREE.MeshStandardMaterial({
    color: 0xffffff,
    emissive: 0xffffff,
    emissiveIntensity: 10
  }),
  100
)
// 随机分布到天上
let starsArr = []
let endArr = []
for (let i = 0; i < 100; i++) {
  let x = Math.random() * 100 - 50
  let y = Math.random() * 100 - 50
  let z = Math.random() * 100 - 50
  starsArr.push(new THREE.Vector3(x, y, z))

  // 设置矩阵
  let matrix = new THREE.Matrix4()
  matrix.setPosition(x, y, z)
  starsInstance.setMatrixAt(i, matrix)
}
scene.add(starsInstance)

// 贝赛尔曲线创建爱心的路线
// let heartShape = new THREE.Shape()
// heartShape.moveTo(25, 25)
// heartShape.bezierCurveTo(25, 25, 20, 0, 0, 0)
// heartShape.bezierCurveTo(-30, 0, -30, 35, -30, 35)
// heartShape.bezierCurveTo(-30, 55, -10, 77, 25, 95)
// heartShape.bezierCurveTo(60, 77, 80, 55, 80, 35)
// heartShape.bezierCurveTo(80, 35, 80, 0, 50, 0)
// heartShape.bezierCurveTo(35, 0, 25, 25, 25, 25)

const x = 0,
  y = 0
const heartRadius = 30
const c = 0.551915024494 * heartRadius

const heartShape = new THREE.Shape()
  .moveTo(x, y + heartRadius / 3)
  .bezierCurveTo(x + c, y + heartRadius, x + heartRadius, y + c, x + heartRadius, y)
  .bezierCurveTo(x + heartRadius, y - c, x + c, y - heartRadius / 2, x, y - heartRadius)
  .bezierCurveTo(x - c, y - heartRadius / 2, x - heartRadius, y - c, x - heartRadius, y)
  .bezierCurveTo(x - heartRadius, y + c, x - c, y + heartRadius, x, y + heartRadius / 3)

// 根据爱心路径获取点
let center = new THREE.Vector3(0, 4, 10)
for (let i = 0; i < 100; i++) {
  let point = heartShape.getPoint(i / 100)
  // endArr.push(new THREE.Vector3(point.x, point.y, point.z))
  endArr.push(new THREE.Vector3(point.x * 0.1 + center.x, point.y * 0.1 + center.y, center.z))
}
// 创建爱心动画
function makeHeart() {
  let params = {
    time: 0
  }
  gsap.to(params, {
    time: 1,
    duration: 1,
    onUpdate: () => {
      for (let i = 0; i < 100; i++) {
        let x = starsArr[i].x + (endArr[i].x - starsArr[i].x) * params.time
        let y = starsArr[i].y + (endArr[i].y - starsArr[i].y) * params.time
        let z = starsArr[i].z + (endArr[i].z - starsArr[i].z) * params.time
        let matrix = new THREE.Matrix4()
        matrix.setPosition(x, y, z)
        starsInstance.setMatrixAt(i, matrix)
      }
      starsInstance.instanceMatrix.needsUpdate = true
    }
  })
}

function restoreHeart() {
  let params = {
    time: 0
  }
  gsap.to(params, {
    time: 1,
    duration: 1,
    onUpdate: () => {
      for (let i = 0; i < 100; i++) {
        let x = endArr[i].x + (starsArr[i].x - endArr[i].x) * params.time
        let y = endArr[i].y + (starsArr[i].y - endArr[i].y) * params.time
        let z = endArr[i].z + (starsArr[i].z - endArr[i].z) * params.time
        let matrix = new THREE.Matrix4()
        matrix.setPosition(x, y, z)
        starsInstance.setMatrixAt(i, matrix)
      }
      starsInstance.instanceMatrix.needsUpdate = true
    }
  })
}
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
