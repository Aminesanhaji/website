<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { TGALoader } from "three/examples/jsm/loaders/TGALoader.js";
// import Robot from "@/public/models/SambaDancing.fbx";

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
        45,
        window.innerWidth / window.innerHeight,
        1,
        2000
      );
      camera.position.set(100, 200, 300);

      scene = new THREE.Scene();
      scene.background = new THREE.Color(0xa0a0a0);
      scene.fog = new THREE.Fog(0xffffff, 2, 1000);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff);
      hemiLight.position.set(10, 200, 10);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff);
      dirLight.position.set(0, 200, 100);
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 180;
      dirLight.shadow.camera.bottom = -100;
      dirLight.shadow.camera.left = -120;
      dirLight.shadow.camera.right = 120;
      scene.add(dirLight);

      // scene.add(new THREE.CameraHelper(dirLight.shadow.camera));
      const tloader = new THREE.TextureLoader();
      const grass = tloader.load(require("@/assets/grass.jpg"));
      // ground
      var groundMaterial = new THREE.MeshToonMaterial({
        // color: 0x000000,
        map: grass,
      });
      var geometry = new THREE.CylinderGeometry(100, 100, 5, 20);
      var mesh = new THREE.Mesh(geometry, groundMaterial);
      mesh.position.y = 0.5;
      mesh.rotation.x += 0.0;
      mesh.rotation.y += 0.02;

      // mesh.rotation.x = -Math.PI / 2;
      mesh.receiveShadow = true;
      scene.add(mesh);

      const grid = new THREE.GridHelper(2000, 20, 0x000000, 0x000000);
      grid.material.opacity = 0.2;
      grid.material.transparent = true;
      scene.add(grid);

      // model

      const manager = new THREE.LoadingManager();
      manager.addHandler(/\.tga$/i, new TGALoader());

      const loader = new FBXLoader(manager);
      // const path = require("./models/fbx/Walking.fbx");
      loader.load("./models/fbx/Waving.fbx", function (object) {
        mixer = new THREE.AnimationMixer(object);

        const action = mixer.clipAction(object.animations[0]);
        action.play();

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

      if (mixer) mixer.update(delta);

      renderer.render(scene, camera);

      stats.update();
    }
  },
};
</script>
