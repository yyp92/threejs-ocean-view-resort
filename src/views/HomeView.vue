<template>
  <div class="home" ref="screenDom">
    <div class="canvas-container"></div>
    <!-- <BigScreen></BigScreen> -->
    <!-- <div class="loading" v-if="progress != 100"></div>
    <div class="progress" v-if="progress != 100">
      <img src="../assets/loading.gif" alt="" />
      <span>酒店加载中：{{ progress }}%</span>
    </div>
    <div class="title">海景酒店日夜交替展示</div> -->
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import ThreePlus from "../three/index";
import * as THREE from "three";
import eventHub from "@/utils/eventHub";
import modifyCityMaterial from "../three/modify/modifyCityMaterial";
import BigScreen from "../components/BigScreen.vue";
import gsap from "gsap";

let progress = ref(0);

let screenDom = ref(null);
const resizeFn = () => {
  // console.log(screenDom);
  let scale = window.innerWidth / 1920;
  screenDom.value.style.transform = `scale(${scale})`;
};
onMounted(() => {
  resizeFn();
  window.addEventListener("resize", resizeFn);
});

onMounted(() => {
  const container = document.querySelector(".canvas-container");
  let threePlus = new ThreePlus(".canvas-container");
  window.threePlus = threePlus;
  threePlus.camera.position.set(-117, 17, -140);

  let bgPromise = threePlus.setBg("./assets/textures/023.hdr");
  threePlus.addOcean();

  // 创建视频纹理
  let video = document.createElement("video");
  video.src = "./textures/video/sucai01.mp4";
  video.autoplay = true;
  video.loop = true;
  video.muted = true;
  video.play();
  let videoTexture = new THREE.VideoTexture(video);

  threePlus.gltfLoader("./model/building-min02.glb").then((gltf) => {
    let vetroMaterial = null;
    gltf.scene.traverse((child) => {
      // 所有物体投射阴影和接收阴影
      if (child.isMesh) {
        child.castShadow = true;
        child.receiveShadow = true;

        // 去除阴影的多余的波纹
        child.material.shadowSide = THREE.BackSide;
      }

      if (child.isMesh && child.name == "Plane") {
        child.visible = false;
      }

      if (
        child.isMesh &&
        child.material.name == "Vetro" &&
        vetroMaterial == null
      ) {
        vetroMaterial = child.material;
      }
    });

    bgPromise.then((texture) => {
      threePlus.addSphereSky(
        () => {
          vetroMaterial.emissive = new THREE.Color(0x000000);
          threePlus.unrealBloomPass.enabled = false;
        },
        () => {
          console.log("夜晚");
          vetroMaterial.emissive = new THREE.Color(0x99cc99);
          vetroMaterial.emissiveMap = videoTexture;
          vetroMaterial.emissiveIntensity = 1;
          threePlus.unrealBloomPass.enabled = true;
        }
      );
    });

    threePlus.scene.add(gltf.scene);
  });

  // THREE.DefaultLoadingManager.onProgress = function (item, loaded, total) {
  //   console.log(item, loaded, total);
  //   progress.value = new Number((loaded / total) * 100).toFixed(2);
  // };
});
</script>

<style>
.home {
  width: 1920px;
  height: 1080px;
  transform-origin: 0 0;
}
.canvas-container {
  width: 100%;
  height: 100%;
}
.loading {
  position: fixed;
  top: 0;
  left: 0;
  width: 1920px;
  height: 1080px;
  background-image: url(../assets/loading.jpg);
  background-size: cover;
  filter: blur(50px);
  z-index: 100;
}
.progress {
  position: fixed;
  top: 0;
  left: 0;
  width: 1920px;
  height: 1080px;
  z-index: 101;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  color: #fff;
}
.progress > img {
  padding: 0 15px;
}

.title {
  width: 380px;
  height: 40px;
  position: fixed;
  right: 100px;
  top: 50px;
  background-color: rgba(0, 0, 0, 0.5);
  line-height: 40px;
  text-align: center;
  color: #fff;
  border-radius: 5px;
  z-index: 110;
}
</style>
