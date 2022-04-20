<template>
  <body>
    <div id="blocker">
      <div id="instructions">
        <p style="font-size: 36px">Click to play</p>
        <p>
          Mouvements: ZQSD<br />
          Saut: SPACE<br />
          Caméra: MOUSE
        </p>
      </div>
    </div>
  </body>
</template>

<script>
import * as THREE from "three";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";
import { TGALoader } from "three/examples/jsm/loaders/TGALoader.js";
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls.js";

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

    init();
    animate();

    function init() {
      const container = document.createElement("div");
      document.body.appendChild(container);

      camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        1,
        10000
      );
      // const helper = new THREE.CameraHelper(camera);

      camera.position.set(75, 250, 1000);

      // console.log(camera.position);

      scene = new THREE.Scene();
      // scene.add(helper);
      scene.background = new THREE.Color(0x77b5fe);
      // scene.fog = new THREE.Fog(0xffffff, 2, 1000);

      const hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.1);
      // hemiLight.color.setHSL(0.6, 1, 0.6);
      // hemiLight.groundColor.setHSL( 0.095, 1, 0.75 );
      hemiLight.position.set(0, 50, 0);
      scene.add(hemiLight);

      const dirLight = new THREE.DirectionalLight(0xffffff, 1);
      dirLight.color.setHSL(0.1, 1, 0.95);
      dirLight.position.set(-1, 1.75, 1);
      dirLight.position.multiplyScalar(30);
      scene.add(dirLight);

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

      // objects
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
      loader.load("/models/fbx/street2.fbx", function (object) {
        object.traverse(function (child) {
          if (child.isMesh) {
            child.castShadow = true;
            child.receiveShadow = true;
          }
        });

        scene.add(object);
      });
      //

      renderer = new THREE.WebGLRenderer({ antialias: true });
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

        velocity.x -= velocity.x * 10.0 * 0.01; // X : gauche et droite
        velocity.z -= velocity.z * 10.0 * 0.01; // Z : Avancer     Y : Saut

        velocity.y -= 9.8 * 100.0 * delta; // 100.0 = mass

        direction.z = Number(moveForward) - Number(moveBackward);
        direction.x = Number(moveRight) - Number(moveLeft);
        direction.normalize(); // this ensures consistent movements in all directions

        if (moveForward || moveBackward)
          velocity.z -= direction.z * 400.0 * delta * 8;
        if (moveLeft || moveRight)
          velocity.x -= direction.x * 400.0 * delta * 8;

        if (onObject === true) {
          velocity.y = Math.max(0, velocity.y);
          canJump = true;
        }
        console.log(velocity.x, velocity.y, velocity.z);
        controls.moveRight(-velocity.x * delta * 2);
        controls.moveForward(-velocity.z * delta * 2);

        // controls.getObject().position.y += velocity.y * delta * 20; // new behavior

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
