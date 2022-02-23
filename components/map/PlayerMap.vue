<template>
  <div id="container">
    <div id="blocker">
      <div id="instructions">
        <p style="font-size: 36px">Click to play</p>
        <p>
          Mouvements: QZDS<br />
          Saut: SPACE<br />
          caméra: SOURIS
        </p>
      </div>
    </div>
    <canvas class="mapp"></canvas>
  </div>
</template>

<script>
import * as THREE from "three";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { TGALoader } from "three/examples/jsm/loaders/TGALoader.js";
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";
import * as SkeletonUtils from "three/examples/jsm/utils/SkeletonUtils.js";

export default {
  mounted() {
    let camera, scene, renderer, controls;

    const objects = [];

    let raycaster;

    let moveForward = false;
    let moveBackward = false;
    let moveLeft = false;
    let moveRight = false;
    let canJump = false;

    let prevTime = performance.now();
    const velocity = new THREE.Vector3();
    const direction = new THREE.Vector3();
    const vertex = new THREE.Vector3();
    const color = new THREE.Color();

    init();
    animate();

    function init() {
      camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        1,
        1000
      );
      camera.position.y = 10;

      scene = new THREE.Scene();
      scene.background = new THREE.Color("#0F193B");
      scene.fog = new THREE.Fog(0xffffff, 0, 750);

      const light = new THREE.HemisphereLight(0xeeeeff, 0x777788, 0.75);
      light.position.set(0.5, 1, 0.75);
      scene.add(light);

      controls = new PointerLockControls(camera, document.body);

      const blocker = document.getElementById("blocker");
      const instructions = document.getElementById("instructions");

      instructions.addEventListener("click", function () {
        controls.lock();
      });

      controls.addEventListener("lock", function () {
        instructions.style.display = "none";
        blocker.style.display = "none";
      });

      controls.addEventListener("unlock", function () {
        blocker.style.display = "block";
        instructions.style.display = "";
      });

      scene.add(controls.getObject());

      const onKeyDown = function (event) {
        switch (event.code) {
          case "ArrowUp":
          case "KeyW":
            moveForward = true;
            break;

          case "ArrowLeft":
          case "KeyA":
            moveLeft = true;
            break;

          case "ArrowDown":
          case "KeyS":
            moveBackward = true;
            break;

          case "ArrowRight":
          case "KeyD":
            moveRight = true;
            break;

          case "Space":
            if (canJump === true) velocity.y += 350;
            canJump = false;
            break;
        }
      };

      const onKeyUp = function (event) {
        switch (event.code) {
          case "ArrowUp":
          case "KeyW":
            moveForward = false;
            break;

          case "ArrowLeft":
          case "KeyA":
            moveLeft = false;
            break;

          case "ArrowDown":
          case "KeyS":
            moveBackward = false;
            break;

          case "ArrowRight":
          case "KeyD":
            moveRight = false;
            break;
        }
      };

      document.addEventListener("keydown", onKeyDown);
      document.addEventListener("keyup", onKeyUp);

      raycaster = new THREE.Raycaster(
        new THREE.Vector3(),
        new THREE.Vector3(0, -1, 0),
        0,
        10
      );

      // floor

      let floorGeometry = new THREE.PlaneGeometry(2000, 2000, 100, 100);
      floorGeometry.rotateX(-Math.PI / 2);

      // vertex displacement

      let position = floorGeometry.attributes.position;

      for (let i = 0, l = position.count; i < l; i++) {
        vertex.fromBufferAttribute(position, i);

        vertex.x += Math.random() * 20 - 10;
        vertex.y += Math.random() * 2;
        vertex.z += Math.random() * 20 - 10;

        position.setXYZ(i, vertex.x, vertex.y, vertex.z);
      }

      floorGeometry = floorGeometry.toNonIndexed(); // ensure each face has unique vertices

      position = floorGeometry.attributes.position;
      const colorsFloor = [];

      for (let i = 0, l = position.count; i < l; i++) {
        color.setHSL(
          Math.random() * 0.3 + 0.5,
          0.75,
          Math.random() * 0.25 + 0.75
        );
        colorsFloor.push(color.r, color.g, color.b);
      }

      floorGeometry.setAttribute(
        "color",
        new THREE.Float32BufferAttribute(colorsFloor, 4)
      );
      const tloader = new THREE.TextureLoader();
      const grass = tloader.load(require("@/assets/grass.jpg"));
      const floorMaterial = new THREE.MeshPhongMaterial({
        vertexColors: true,
        map: grass,
      });

      const floor = new THREE.Mesh(floorGeometry, floorMaterial);
      scene.add(floor);

      // FBX 3D
      const manager = new THREE.LoadingManager();
      manager.addHandler(/\.tga$/i, new TGALoader());

      const loader = new FBXLoader(manager);
      loader.load("/models/fbx/tree.fbx", function (object) {
        object.scale.multiplyScalar(0.08);
        object.traverse(function (child) {
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });
        const model1 = SkeletonUtils.clone(object);
        // const model2 = SkeletonUtils.clone(object);
        // const model3 = SkeletonUtils.clone(object);

        model1.position.x = Math.floor(Math.random() * 20 - 10) * 20;
        // model2.position.x = Math.floor(Math.random() * 20 - 10) * 20;
        // model3.position.x = Math.floor(Math.random() * 20 - 10) * 20;

        scene.add(model1);
      });

      //
      // Canvas
      const canvas = document.querySelector("canvas.mapp");

      renderer = new THREE.WebGLRenderer({
        antialias: true,
        canvas: canvas,
      });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      //

      window.addEventListener("resize", onWindowResize);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
      requestAnimationFrame(animate);

      const time = performance.now();

      if (controls.isLocked === true) {
        raycaster.ray.origin.copy(controls.getObject().position);
        raycaster.ray.origin.y -= 10;

        const intersections = raycaster.intersectObjects(objects, false);

        const onObject = intersections.length > 0;

        const delta = (time - prevTime) / 1000;

        velocity.x -= velocity.x * 10.0 * delta;
        velocity.z -= velocity.z * 10.0 * delta;

        velocity.y -= 9.8 * 100.0 * delta; // 100.0 = mass

        direction.z = Number(moveForward) - Number(moveBackward);
        direction.x = Number(moveRight) - Number(moveLeft);
        direction.normalize(); // this ensures consistent movements in all directions

        if (moveForward || moveBackward)
          velocity.z -= direction.z * 400.0 * delta;
        if (moveLeft || moveRight) velocity.x -= direction.x * 400.0 * delta;

        if (onObject === true) {
          velocity.y = Math.max(0, velocity.y);
          canJump = true;
        }

        controls.moveRight(-velocity.x * delta);
        controls.moveForward(-velocity.z * delta);

        controls.getObject().position.y += velocity.y * delta; // new behavior

        if (controls.getObject().position.y < 10) {
          velocity.y = 0;
          controls.getObject().position.y = 10;

          canJump = true;
        }
      }

      prevTime = time;

      renderer.render(scene, camera);
    }
  },
};
</script>

<style>
#blocker {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

#instructions {
  width: 100%;
  height: 100%;

  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  text-align: center;
  font-size: 14px;
  cursor: pointer;
}
</style>
