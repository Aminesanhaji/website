<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from "three";
import gsap from "gsap";
// import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import Stats from "three/examples/jsm/libs/stats.module.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { TGALoader } from "three/examples/jsm/loaders/TGALoader.js";

export default {
  mounted() {
    let camera, scene, renderer, stats;

    const clock = new THREE.Clock();

    let mixer;

    init();
    animate();

    function init() {
      const container = document.createElement("div");
      document.body.appendChild(container);

      camera = new THREE.PerspectiveCamera(
        70,
        window.innerWidth / window.innerHeight,
        1,
        10000
      );
      camera.position.set(0, 250, 1000);
      // camera.position.y = 10;
      // camera.position.z = 20;

      scene = new THREE.Scene();
      scene.background = new THREE.Color(0xffffffff);
      // scene.fog = new THREE.Fog(0xffffff, 2, 1000);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.6);
      hemiLight.color.setHSL(0.6, 1, 0.6);
      // hemiLight.groundColor.setHSL( 0.095, 1, 0.75 );
      hemiLight.position.set(0, 50, 0);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff, 1);
      dirLight.color.setHSL(0.1, 1, 0.95);
      dirLight.position.set(-1, 1.75, 1);
      dirLight.position.multiplyScalar(30);
      scene.add(dirLight);

      // const dirLight = new THREE.DirectionalLight(0xffffff);
      // dirLight.position.set(0, 200, 100);
      // dirLight.castShadow = true;
      // dirLight.shadow.camera.top = 180;
      // dirLight.shadow.camera.bottom = -100;
      // dirLight.shadow.camera.left = -120;
      // dirLight.shadow.camera.right = 120;
      // scene.add(dirLight);

      // scene.add(new THREE.CameraHelper(dirLight.shadow.camera));
      const tloader = new TGALoader();

      // ground
      const grid = new THREE.GridHelper(2000, 20, 0x000000, 0x000000);
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      scene.add(grid);

      // model
      const manager = new THREE.LoadingManager();
      manager.addHandler(/\.tga$/i, new TGALoader());

      const loader = new FBXLoader(manager);
      // const path = require("./models/fbx/Walking.fbx");
      loader.load("/models/fbx/street.fbx", function (object) {
        mixer = new THREE.AnimationMixer(object);

        // const action = mixer.clipAction(object.animations[0]);
        // action.play();

        object.traverse(function (child) {
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });

        scene.add(object);
      });

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.shadowMap.enabled = true;
      container.appendChild(renderer.domElement);

      const controls = new OrbitControls(camera, renderer.domElement);
      controls.target.set(0, 100, 0);
      controls.update();

      window.addEventListener("resize", onWindowResize);

      // stats
      stats = new Stats();
      container.appendChild(stats.dom);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    //

    function animate() {
      requestAnimationFrame(animate);

      const delta = clock.getDelta();

      renderer.render(scene, camera);

      stats.update();
    }
  },
};
</script>

<style></style>
