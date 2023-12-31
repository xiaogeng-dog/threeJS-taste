<template>
  <div class="canvasContainer" ref="canvasContainer"></div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const canvasContainer = ref(null)
onMounted(() => {
  // 初始化场景
  const scene = new THREE.Scene()
  // 初始化相机
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  // 初始化相机位置
  camera.position.set(1.5, 1, 1.5)
  camera.aspect = window.innerWidth / window.innerHeight
  // 更新相机投影矩阵
  camera.updateProjectionMatrix()

  // 加载背景纹理
  const loader = new THREE.TextureLoader()
  const bgTexture = loader.load('/bear/imgs/050.jpg')
  //光折射
  bgTexture.mapping = THREE.EquirectangularRefractionMapping

  scene.background = bgTexture
  scene.environment = bgTexture
  // 添加环境光
  const ambient = new THREE.AmbientLight(0xffffff, 1)
  scene.add(ambient)

  // 加载小熊模型
  const gltfLoader = new GLTFLoader()
  gltfLoader.load('/bear/model/bear.gltf', (gltf) => {
    // console.log(gltf);
    const model = gltf.scene.children[0]
    model.material = new THREE.MeshPhongMaterial({
      color: 0xffffff,
      envMap: bgTexture,
      refractionRatio: 0.7,
      reflectivity: 0.99
    })

    scene.add(model)
  })

  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true })
  // 设置渲染器大小
  renderer.setSize(window.innerWidth, window.innerHeight)
  // 监听屏幕大小变化
  window.addEventListener('resize', () => {
    renderer.setSize(window.innerWidth, window.innerHeight)
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
  })

  // 将渲染器添加到页面
  canvasContainer.value.appendChild(renderer.domElement)

  // 添加控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  // 设置控制器阻尼
  controls.enableDamping = true

  // 渲染函数
  const render = () => {
    // 更新控制器
    controls.update()
    // 渲染场景
    renderer.render(scene, camera)
    // 循环渲染
    requestAnimationFrame(render)
  }

  render()
})
</script>

<style scoped lang="less">
* {
  margin: 0;
  padding: 0;
}
</style>
