<template>
  <div class="mainn">
    <canvas class="webgll"></canvas>
    <section class="section">
      <h1>My Portfolio</h1>
    </section>
    <section class="section">
      <h2>My projects</h2>
    </section>
    <section class="section">
      <h2>Contact me</h2>
    </section>
  </div>
</template>

<script>
import * as THREE from "three";
import gsap from "gsap";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";
export default {
  mounted() {
    /**
     * Base
     */

    const loader = new THREE.TextureLoader();
    const cross = loader.load(require("@/assets/cross0.png"));
    const stars = loader.load(require("@/assets/cross1.png"));

    // Canvas
    const canvas = document.querySelector("canvas.webgll");

    // Scene
    const scene = new THREE.Scene();

    // colors
    let colorsArray = [
      "63b598",
      "ce7d78",
      "ea9e70",
      "a48a9e",
      "c6e1e8",
      "648177",
      "0d5ac1",
      "f205e6",
      "1c0365",
      "14a9ad",
      "4ca2f9",
      "a4e43f",
      "d298e2",
      "6119d0",
      "d2737d",
      "c0a43c",
      "f2510e",
      "651be6",
      "79806e",
      "61da5e",
      "cd2f00",
      "9348af",
      "01ac53",
      "c5a4fb",
      "996635",
      "b11573",
      "4bb473",
      "75d89e",
      "2f3f94",
      "2f7b99",
      "da967d",
      "34891f",
      "b0d87b",
      "ca4751",
      "7e50a8",
      "c4d647",
      "e0eeb8",
      "11dec1",
      "289812",
      "566ca0",
      "ffdbe1",
      "2f1179",
      "935b6d",
      "916988",
      "513d98",
      "aead3a",
      "9e6d71",
      "4b5bdc",
      "0cd36d",
      "250662",
      "cb5bea",
      "228916",
      "ac3e1b",
      "df514a",
      "539397",
      "880977",
      "f697c1",
      "ba96ce",
      "679c9d",
      "c6c42c",
      "5d2c52",
      "48b41b",
      "e1cf3b",
      "5be4f0",
      "57c4d8",
      "a4d17a",
      "225b8",
      "be608b",
      "96b00c",
      "088baf",
      "f158bf",
      "e145ba",
      "ee91e3",
      "05d371",
      "5426e0",
      "4834d0",
      "802234",
      "6749e8",
      "0971f0",
      "8fb413",
      "b2b4f0",
      "c3c89d",
      "c9a941",
      "41d158",
      "fb21a3",
      "51aed9",
      "5bb32d",
      "807fb",
      "21538e",
      "89d534",
      "d36647",
      "7fb411",
      "0023b8",
      "3b8c2a",
      "986b53",
      "f50422",
      "983f7a",
      "ea24a3",
      "79352c",
      "521250",
      "c79ed2",
      "d6dd92",
      "e33e52",
      "b2be57",
      "fa06ec",
      "1bb699",
      "6b2e5f",
      "64820f",
      "1c271",
      "21538e",
      "89d534",
      "d36647",
      "7fb411",
      "0023b8",
      "3b8c2a",
      "986b53",
      "f50422",
      "983f7a",
      "ea24a3",
      "79352c",
      "521250",
      "c79ed2",
      "d6dd92",
      "e33e52",
      "b2be57",
      "fa06ec",
      "1bb699",
      "6b2e5f",
      "64820f",
      "1c271",
      "9cb64a",
      "996c48",
      "9ab9b7",
      "06e052",
      "e3a481",
      "0eb621",
      "fc458e",
      "b2db15",
      "aa226d",
      "792ed8",
      "73872a",
      "520d3a",
      "cefcb8",
      "a5b3d9",
      "7d1d85",
      "c4fd57",
      "f1ae16",
      "8fe22a",
      "ef6e3c",
      "243eeb",
      "1dc18",
      "dd93fd",
      "3f8473",
      "e7dbce",
      "421f79",
      "7a3d93",
      "635f6d",
      "93f2d7",
      "9b5c2a",
      "15b9ee",
      "0f5997",
      "409188",
      "911e20",
      "1350ce",
      "10e5b1",
      "fff4d7",
      "cb2582",
      "ce00be",
      "32d5d6",
      "17232",
      "608572",
      "c79bc2",
      "00f87c",
      "77772a",
      "6995ba",
      "fc6b57",
      "f07815",
      "8fd883",
      "060e27",
      "96e591",
      "21d52e",
      "d00043",
      "b47162",
      "1ec227",
      "4f0f6f",
      "1d1d58",
      "947002",
      "bde052",
      "e08c56",
      "28fcfd",
      "bb09b",
      "36486a",
      "d02e29",
      "1ae6db",
      "3e464c",
      "a84a8f",
      "911e7e",
      "3f16d9",
      "0f525f",
      "ac7c0a",
      "b4c086",
      "c9d730",
      "30cc49",
      "3d6751",
      "fb4c03",
      "640fc1",
      "62c03e",
      "d3493a",
      "88aa0b",
      "406df9",
      "615af0",
      "4be47",
      "2a3434",
      "4a543f",
      "79bca0",
      "a8b8d4",
      "00efd4",
      "7ad236",
      "7260d8",
      "1deaa7",
      "06f43a",
      "823c59",
      "e3d94c",
      "dc1c06",
      "f53b2a",
      "b46238",
      "2dfff6",
      "a82b89",
      "1a8011",
      "436a9f",
      "1a806a",
      "4cf09d",
      "c188a2",
      "67eb4b",
      "b308d3",
      "fc7e41",
      "af3101",
      "ff065",
      "71b1f4",
      "a2f8a5",
      "e23dd0",
      "d3486d",
      "00f7f9",
      "474893",
      "3cec35",
      "1c65cb",
      "5d1d0c",
      "2d7d2a",
      "ff3420",
      "5cdd87",
      "a259a4",
      "e4ac44",
      "1bede6",
      "8798a4",
      "d7790f",
      "b2c24f",
      "de73c2",
      "d70a9c",
      "25b67",
      "88e9b8",
      "c2b0e2",
      "86e98f",
      "ae90e2",
      "1a806b",
      "436a9e",
      "0ec0ff",
      "f812b3",
      "b17fc9",
      "8d6c2f",
      "d3277a",
      "2ca1ae",
      "9685eb",
      "8a96c6",
      "dba2e6",
      "76fc1b",
      "608fa4",
      "20f6ba",
      "07d7f6",
      "dce77a",
      "77ecca",
    ];
    const colour = new THREE.Color();
    colour.setHex(
      `0x${colorsArray[Math.floor(Math.random() * colorsArray.length)]}`
    );
    if (colour < 500) {
      colour.setHex(500);
    }

    /**
     * Objects
     */
    const objectsDistance = 4;

    // Material
    const material = new THREE.PointsMaterial({
      color: colour,
      size: 0.1,
      // map: stars,
      transparent: true,
      rotation: 2,
    });

    // OBJ model

    const obj = new OBJLoader();
    obj.load("/models/fbx/globe2.obj", function (object) {
      let material = new THREE.PointsMaterial({
        color: colour,
        size: 0.035,
        map: cross,
      });
      let mesh = new THREE.Points(object.children[0].geometry, material);

      mesh.scale.multiplyScalar(0.025);
      // mesh.rotation.y = 2 * Math.PI * Math.random();
      mesh.position.x = objectsDistance * 0.5;
      mesh.position.y = -objectsDistance * 0.2;
      scene.add(mesh);
    });
    //----------------------

    // Meshes
    const mesh2 = new THREE.Mesh(new THREE.ConeGeometry(1, 2, 32), material);
    const mesh3 = new THREE.Points(
      new THREE.TorusKnotGeometry(0.8, 0.35, 100, 16),
      material
    );

    // mesh1.position.y = -objectsDistance * 0;
    mesh2.position.y = -objectsDistance * 1;
    mesh3.position.y = -objectsDistance * 2;

    // mesh1.position.x = 2;
    mesh2.position.x = -4;
    mesh3.position.x = 3;

    const sectionMeshes = [mesh2, mesh3];

    scene.add(mesh2, mesh3);

    /**
     * Particles
     */
    // Geometry
    const particlesCount = 1000;
    const positions = new Float32Array(particlesCount * 3);

    for (let i = 0; i < particlesCount; i++) {
      positions[i * 3 + 0] = (Math.random() - 0.5) * 10;
      positions[i * 3 + 1] =
        objectsDistance * 0.5 -
        Math.random() * objectsDistance * sectionMeshes.length;
      positions[i * 3 + 2] = (Math.random() - 0.5) * 10;
    }

    const particlesGeometry = new THREE.BufferGeometry();
    particlesGeometry.setAttribute(
      "position",
      new THREE.BufferAttribute(positions, 3)
    );

    // Material
    const particlesMaterial = new THREE.PointsMaterial({
      color: "#c0a43c",
      sizeAttenuation: true,
      size: 0.2,
      transparent: true,
      map: cross,
    });

    // Points
    const particles = new THREE.Points(particlesGeometry, particlesMaterial);
    scene.add(particles);

    //   // Interaction
    var uniforms = {
      mouse: {
        value: new THREE.Vector3(),
      },
      radius: {
        value: 0.5,
      },
    };

    material.onBeforeCompile = (shader) => {
      shader.uniforms.mouse = uniforms.mouse;
      shader.uniforms.radius = uniforms.radius;
      // console.log(shader.vertexShader);
      shader.vertexShader =
        `
    	uniform vec3 mouse;
      uniform float radius;
    ` + shader.vertexShader;
      shader.vertexShader = shader.vertexShader.replace(
        `#include <begin_vertex>`,
        `#include <begin_vertex>

        vec3 dir = transformed - mouse;

        float dist = length(dir);

        if (dist < radius){
          float ratio = 1. - dist / radius;

          vec3 shift = dir * 2. * (ratio * ratio);

          transformed += shift;
        }

      `
      );
    };

    var plane = new THREE.Plane(new THREE.Vector3(0, 0, 1), 0);
    var raycaster = new THREE.Raycaster();
    var mouse = new THREE.Vector2();
    window.addEventListener("mousemove", (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

      raycaster.setFromCamera(mouse, camera);

      raycaster.ray.intersectPlane(plane, uniforms.mouse.value);
    });

    /**
     * Lights
     */
    const directionalLight = new THREE.DirectionalLight("#ffffff", 1);
    directionalLight.position.set(1, 1, 0);
    scene.add(directionalLight);

    /**
     * Sizes
     */
    const sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    window.addEventListener("resize", () => {
      // Update sizes
      sizes.width = window.innerWidth;
      sizes.height = window.innerHeight;

      // Update camera
      camera.aspect = sizes.width / sizes.height;
      camera.updateProjectionMatrix();

      // Update renderer
      renderer.setSize(sizes.width, sizes.height);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    });

    /**
     * Scroll
     */
    let scrollY = window.scrollY;
    let currentSection = 0;

    window.addEventListener("scroll", () => {
      scrollY = window.scrollY;
      const newSection = Math.round(scrollY / sizes.height);

      if (newSection != currentSection) {
        currentSection = newSection;

        gsap.to(sectionMeshes[currentSection].rotation, {
          duration: 1.5,
          ease: "power2.inOut",
          x: "+=10",
          y: "+=7",
        });
      }
    });

    /**
     * Cursor
     */
    const cursor = {};
    cursor.x = 0;
    cursor.y = 0;

    window.addEventListener("mousemove", (event) => {
      cursor.x = event.clientX / sizes.width - 0.5;
      cursor.y = event.clientY / sizes.height - 0.5;
    });

    /**
     * Camera
     */
    // Group
    const cameraGroup = new THREE.Group();
    scene.add(cameraGroup);

    // Base camera
    const camera = new THREE.PerspectiveCamera(
      35,
      sizes.width / sizes.height,
      0.1,
      100
    );
    camera.position.z = 9;
    cameraGroup.add(camera);

    /**
     * Renderer
     */
    const renderer = new THREE.WebGLRenderer({
      canvas: canvas,
      alpha: true,
    });
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    /**
     * Animate
     */
    const clock = new THREE.Clock();
    let previousTime = 0;

    const tick = () => {
      const elapsedTime = clock.getElapsedTime();
      const deltaTime = elapsedTime - previousTime;
      previousTime = elapsedTime;

      // nejma.update(elapsedTime * 1000000);

      // Animate meshes
      for (const mesh of sectionMeshes) {
        mesh.rotation.x += deltaTime * 0.1;
        mesh.rotation.y += deltaTime * 0.12;
      }

      // Animate camera
      camera.position.y = (-scrollY / sizes.height) * objectsDistance;

      const parallaxX = cursor.x * 0.5;
      const parallaxY = -cursor.y * 0.5;

      cameraGroup.position.x +=
        (parallaxX - cameraGroup.position.x) * 5 * deltaTime;
      cameraGroup.position.y +=
        (parallaxY - cameraGroup.position.y) * 5 * deltaTime;

      // Render
      renderer.render(scene, camera);

      // Call tick again on the next frame
      window.requestAnimationFrame(tick);
    };

    tick();
  },
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

.mainn {
  background: #100f11;
}

.webgll {
  position: fixed;
  top: 0;
  left: 0;
}

.section {
  display: flex;
  align-items: center;
  height: 100vh;
  position: relative;
  font-family: "Cabin", sans-serif;
  color: #ffeded;
  text-transform: uppercase;
  font-size: 7vmin;
  padding-left: 10%;
  padding-right: 10%;
  will-change: transform;
}

.art {
  font-family: "Cabin", sans-serif;
  font-size: 20px;
  width: 350px;
}

section:nth-child(odd) {
  justify-content: flex-end;
}
</style>
