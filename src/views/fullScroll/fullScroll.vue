<template>
  <div class="page-container" id="page-container">
    <div class="page">
      <div class="page-title">
        <h1>Box</h1>
        <h3>THREE.BoxGeometry</h3>
      </div>
    </div>
    <div class="page-space"></div>
    <div class="page">
      <div class="page-title">
        <h1>Cylinder</h1>
        <h3>THREE.CylinderGeometry</h3>
      </div>
    </div>
    <div class="page-space"></div>
    <div class="page">
      <div class="page-title">
        <h1>Dodecahedron</h1>
        <h3>THREE.DodecahedronGeometry</h3>
      </div>
    </div>
    <div class="page-space"></div>
  </div>
</template>
<script setup lang="ts">
import * as THREE from 'three'
// 导入轨道控制器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
// 导入动画库
import gsap from 'gsap'
import { onMounted } from 'vue'
// 导入dat.gui
import * as dat from 'dat.gui'
//#region <创建场景>
// 创建场景
const scene = new THREE.Scene()
// scene.background = new THREE.Color(0x000000)
//#endregion
//#region <灯光>
// 创建灯光
const light = new THREE.DirectionalLight(0xffffff, 1)
// 设置灯光位置
light.position.set(0, 0, 30)
// 将灯光添加进场景
scene.add(light)
//#endregion
//#region <相机>
// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  70,
)
// 设置相机位置
// camera.position.set(0, 0, 30)
camera.position.set(0, 0, 30)

scene.add(camera)
//#endregion
//#region <创建物体>
// 位置间距
let spacing = 200
//#region <立方体>
const cubeGeometry = new THREE.BoxGeometry(3, 3, 3)
const cylinderGeometry = new THREE.CylinderGeometry(2, 2, 3, 32)
const dodecahedronGeometry = new THREE.DodecahedronGeometry(2, 5)
const material = new THREE.MeshBasicMaterial({
  wireframe: true,
})
function createMesh(
  Geometry: THREE.BufferGeometry,
  Material: THREE.MeshBasicMaterial,
  translate: { x: number; y: number; z: number } = {
    x: 0,
    y: 0,
    z: 0,
  },
) {
  // 创建无数立方体
  let meshArr: THREE.Mesh<THREE.BufferGeometry, THREE.MeshBasicMaterial>[] = []
  let meshGroup = new THREE.Group()
  for (let i = 0; i < 5; i++) {
    for (let j = 0; j < 5; j++) {
      for (let z = 0; z < 5; z++) {
        const cube = new THREE.Mesh(Geometry, Material)
        cube.position.set(
          i * 3 - 6 + (translate.x || 0),
          j * 3 - 6 + (translate.y || 0),
          z * 3 - 6 + (translate.z || 0),
        )
        meshGroup.add(cube)
        meshArr.push(cube)
      }
    }
  }
  scene.add(meshGroup)
  return {
    meshArr,
    meshGroup,
  }
}
let { meshArr: cubeArr, meshGroup: cubeGroup } = createMesh(
  cubeGeometry,
  material,
)
let { meshArr: cylinderArr, meshGroup: cylinderGroup } = createMesh(
  cylinderGeometry,
  material,
  {
    x: 0,
    y: -spacing,
    z: 0,
  },
)
let { meshArr: dodecahedronrArr, meshGroup: dodecahedronrGroup } = createMesh(
  dodecahedronGeometry,
  material,
  {
    x: 0,
    y: -2 * spacing,
    z: 0,
  },
)
let meshArr = [cubeArr, cylinderArr, dodecahedronrArr]
let meshGroup = [cubeGroup, cylinderGroup, dodecahedronrGroup]
//#endregion
//#region <物体动画>
meshGroup.forEach((item) => {
  gsap.to(item.rotation, {
    x: 0,
    y: Math.PI,
    duration: 5,
    ease: 'linear',
    // yoyo: true,
    repeat: -1,
  })
})

//#endregion
//#endregion
//#region <平面>
// 在y:-100的位置添加一个平面遮挡
// function creatPlame(
//   translate: { x?: number; y?: number; z?: number } = {
//     x: 0,
//     y: 0,
//     z: 0,
//   },
// ) {
//   const planeGeometry = new THREE.PlaneGeometry(1000, 1000)
//   const planeMaterial = new THREE.MeshBasicMaterial({
//     color: '#88CE02',
//     side: THREE.DoubleSide,
//   })
//   const plane = new THREE.Mesh(planeGeometry, planeMaterial)
//   plane.rotation.x = -Math.PI / 2
//   plane.position.set(translate.x || 0, translate.y || 0, translate.z || 0)
//   scene.add(plane)
// }
// creatPlame({ y: 100 })
// creatPlame({ y: 300 })
// creatPlame({ y: -100 })
// creatPlame({ y: -300 })
//#endregion
//#region <渲染器>
// 初始化渲染器
// 渲染器透明
const renderer = new THREE.WebGLRenderer({ alpha: true })
// 设置渲染的尺寸大小
renderer.setSize(window.innerWidth, window.innerHeight)
// 开启场景中的阴影贴图
renderer.shadowMap.enabled = true
// renderer.physicallyCorrectLights = true
renderer.render(scene, camera)
//#endregion
//#region <坐标轴辅助器>
// 创建坐标轴辅助器
const axesHelper = new THREE.AxesHelper(5)
// 将坐标轴辅助器添加到场景中
scene.add(axesHelper)
//#endregion
//#region <轨道控制器>
// 创建轨道控制器
const controls = new OrbitControls(camera, renderer.domElement)
// 设置控制器阻尼，让控制器更有弹性
// 实现真实效果，必须在动画循环里的每一帧调用.update()。
controls.enableDamping = true
// 禁止缩放
controls.enableZoom = false
//#endregion
//#region <渲染>
// const clock = new THREE.Clock()
// 渲染函数
function render() {
  controls.update()
  renderer.render(scene, camera)
  //   渲染下一帧的时候就会调用render函数
  requestAnimationFrame(render)
}
// 监听画面变化，更新渲染画面
window.addEventListener('resize', () => {
  console.log('画面变化了')

  // 更新摄像头
  camera.aspect = window.innerWidth / window.innerHeight
  //   更新摄像机的投影矩阵
  camera.updateProjectionMatrix()

  //   更新渲染器
  renderer.setSize(window.innerWidth, window.innerHeight)
  //   设置渲染器的像素比
  renderer.setPixelRatio(window.devicePixelRatio)
})
//#endregion
//#region <控制面板>
const gui = new dat.GUI()
gui
  .add(camera.position, 'y')
  .min(-100)
  .max(100)
  .step(0.01)
  .name('移动相机Y位置')
  .onChange((value) => {
    console.log('值被修改：', value)
  })
  .onFinishChange((value) => {
    console.log('完全停下来:', value)
  })
gui
  .add(camera.position, 'x')
  .min(-100)
  .max(100)
  .step(0.01)
  .name('移动相机X位置')
  .onChange((value) => {
    console.log('值被修改：', value)
  })
  .onFinishChange((value) => {
    console.log('完全停下来:', value)
  })
gui
  .add(camera.position, 'z')
  .min(-100)
  .max(100)
  .step(0.01)
  .name('移动相机Z位置')
  .onChange((value) => {
    console.log('值被修改：', value)
  })
  .onFinishChange((value) => {
    console.log('完全停下来:', value)
  })
//#endregion
//#region <监听事件双击、滚动事件>
// 双击事件
window.addEventListener('dblclick', () => {
  const fullScreenElement = document.fullscreenElement
  if (!fullScreenElement) {
    //   双击控制屏幕进入全屏，退出全屏
    // 让画布对象全屏
    renderer.domElement.requestFullscreen()
  } else {
    //   退出全屏，使用document对象
    document.exitFullscreen()
  }
  //   console.log(fullScreenElement);
})
// 单击事件
// 创建投射光线对象
const raycaster = new THREE.Raycaster()
// 鼠标的位置对象
const mouse = new THREE.Vector2()
// 监听鼠标的位置
window.addEventListener('click', (event) => {
  // 将鼠标位置归一化为设备坐标。x 和 y 方向的取值范围是 (-1 to +1)
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -((event.clientY / window.innerHeight) * 2 - 1)
  raycaster.setFromCamera(mouse, camera)
  meshArr.forEach((item) => {
    let intersects = raycaster.intersectObjects(item)
    console.log('intersects ', intersects)
    if (intersects.length === 0) return
    ;(intersects[0].object as unknown as any).material =
      new THREE.MeshBasicMaterial({
        // 粉色
        // 随机颜色
        color: getRandomColor(),
      })
  })

  // intersects.forEach((item) => {
  //   // 重新设置材质
  //   item.object.material = redMaterial
  // })
})
function getRandomColor() {
  const r = Math.floor(Math.random() * 256)
  const g = Math.floor(Math.random() * 256)
  const b = Math.floor(Math.random() * 256)

  const rr = ('0' + r.toString(16)).slice(-2)
  const gg = ('0' + g.toString(16)).slice(-2)
  const bb = ('0' + b.toString(16)).slice(-2)

  return '#' + rr + gg + bb
}
function eventListrenderHandler(element: HTMLElement) {
  // 滚动事件
  element.addEventListener('wheel', (e) => {
    // 页面可视高度
    const innerHeight = window.innerHeight
    // 判断e.deltaY大于0 页面向下滑动 位置坐标向上走
    // y轴移动距离
    let translateY = gsap.utils.clamp(
      -2 * window.innerHeight,
      0,
      +gsap.getProperty('.page', 'y') +
        (-e.deltaY / Math.abs(e.deltaY)) * innerHeight,
    )
    // 更新页面位置
    gsap.to('.page', {
      y: translateY,
      duration: 0.5,
    })
    // 更新物体坐标位置
    let meshTranslateY = gsap.utils.clamp(
      0,
      2 * spacing,
      -(translateY / innerHeight) * spacing,
    )
    meshGroup.forEach((item) => {
      gsap.to(item.position, {
        x: 0,
        y: meshTranslateY,
        z: 0,
        duration: 0.5,
      })
    })
    //  重新设置摄像机位置
    camera.position.set(0, 0, 30)
  })
}
//#endregion
onMounted(() => {
  // 将webgl渲染的canvas内容添加到body中
  let pageContainer = document.getElementById('page-container') || document.body
  pageContainer?.appendChild(renderer.domElement)
  eventListrenderHandler(pageContainer)
  render()
})
</script>
<style lang="scss">
canvas {
  position: fixed;
  left: 0;
  top: 0;
}
</style>
<style scoped lang="scss">
.page-container {
  position: relative;
  background-color: skyblue;
  height: 100vh;
  overflow: hidden;
}
canvas {
  position: fixed;
  left: 0;
  top: 0;
}
.page {
  height: 30vh;
  display: flex;
  color: #fff;
  flex-direction: column;
  /* justify-content: center; */
  align-items: center;
  position: relative;
  z-index: 10;
  overflow: hidden;
}
.page-space {
  height: 70vh;
}
.page-title {
  position: relative;
  z-index: 999;
  h1 {
    margin: 20px 0;
    font-size: 40px;
  }
  h3 {
    font-size: 30px;
  }
}
</style>
