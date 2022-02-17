<template>
  <body>
    <span id="scrollProgress"></span>
    <main>
      <h1>Animate on Scroll</h1>
      <section>
        <h2>Scroll ppur voir un truc sympa</h2>
      </section>
      <section>
        <h2>Changing Objects Position</h2>
        <p>The cubes position is now changing</p>
      </section>

      <section>
        <h2>Le cube tourne ici</h2>
        <p>The cubes rotation is now changing</p>
      </section>

      <section>
        <h2>La poition de caméra change iciii</h2>
        <p>The camera position is now changing</p>
      </section>

      <section>
        <h2>vous êtes tout en bas du coup</h2>
        <p>The cube will now be auto rotating</p>
        <p>Scroll en haut pour inverser l'animation</p>
      </section>
    </main>
  </body>
</template>

<script>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module.js";
export default {
  mounted() {
    var scene = new THREE.Scene();
    var gridHelper = new THREE.GridHelper(10, 10, 0xaec6ff, 0x7fbba6);
    scene.add(gridHelper);
    var camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    var renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);
    var geometry = new THREE.BoxGeometry();
    var material = new THREE.MeshBasicMaterial({
      color: 0x343ab6,
      wireframe: true,
    });
    var cube = new THREE.Mesh(geometry, material);
    cube.position.set(0, 0.5, -10);
    scene.add(cube);
    window.addEventListener("resize", onWindowResize, false);
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
      render();
    }
    /* Liner Interpolation
     * lerp(min, max, ratio)
     * eg,
     * lerp(20, 60, .5)) = 40
     * lerp(-20, 60, .5)) = 20
     * lerp(20, 60, .75)) = 50
     * lerp(-20, -10, .1)) = -.19
     */
    function lerp(x, y, a) {
      return (1 - a) * x + a * y;
    }
    // Used to fit the lerps to start and end at specific scrolling percentages
    function scalePercent(start, end) {
      return (scrollPercent - start) / (end - start);
    }
    var animationScripts = [];
    //add an animation that flashes the cube through 100 percent of scroll
    animationScripts.push({
      start: 0,
      end: 101,
      func: function () {
        var g = material.color.g;
        g -= 0.05;
        if (g <= 0) {
          g = 1.0;
        }
        material.color.g = g;
      },
    });
    //add an animation that moves the cube through first 40 percent of scroll
    animationScripts.push({
      start: 0,
      end: 40,
      func: function () {
        camera.lookAt(cube.position);
        camera.position.set(0, 1, 2);
        cube.position.z = lerp(-10, 0, scalePercent(0, 40));
        //console.log(cube.position.z)
      },
    });
    //add an animation that rotates the cube between 40-60 percent of scroll
    animationScripts.push({
      start: 40,
      end: 60,
      func: function () {
        camera.lookAt(cube.position);
        camera.position.set(0, 1, 2);
        cube.rotation.z = lerp(0, Math.PI, scalePercent(40, 60));
        //console.log(cube.rotation.z)
      },
    });
    //add an animation that moves the camera between 60-80 percent of scroll
    animationScripts.push({
      start: 60,
      end: 80,
      func: function () {
        camera.position.x = lerp(0, 5, scalePercent(60, 80));
        camera.position.y = lerp(1, 5, scalePercent(60, 80));
        camera.lookAt(cube.position);
        //console.log(camera.position.x + " " + camera.position.y)
      },
    });
    //add an animation that auto rotates the cube from 80 percent of scroll
    animationScripts.push({
      start: 80,
      end: 101,
      func: function () {
        //auto rotate
        cube.rotation.x += 0.01;
        cube.rotation.y += 0.01;
      },
    });
    function playScrollAnimations() {
      animationScripts.forEach(function (a) {
        if (scrollPercent >= a.start && scrollPercent < a.end) {
          a.func();
        }
      });
    }
    var scrollPercent = 0;
    document.body.onscroll = function () {
      //calculate the current scroll progress as a percentage
      scrollPercent =
        ((document.documentElement.scrollTop || document.body.scrollTop) /
          ((document.documentElement.scrollHeight ||
            document.body.scrollHeight) -
            document.documentElement.clientHeight)) *
        100;
      document.getElementById("scrollProgress").innerText =
        "Scroll Progress : " + scrollPercent.toFixed(2);
    };
    var stats = Stats();
    document.body.appendChild(stats.dom);
    function animate() {
      requestAnimationFrame(animate);
      playScrollAnimations();
      render();
      stats.update();
    }
    function render() {
      renderer.render(scene, camera);
    }
    window.scrollTo({ top: 0, behavior: "smooth" });
    animate();
  },
};
</script>

<style>
body {
  overflow-x: hidden;
  margin: 0px;
  font-family: monospace;
  color: white;
}

canvas {
  position: fixed;
  top: 0;
  left: 0;
}

main {
  width: 100vw;
  height: 200vw;
  z-index: 99;
  position: absolute;
  justify-content: center;
  text-align: center;
  pointer-events: none;
  font-size: 5vh;
}

section {
  min-height: 100vh;
  padding: 20px;
  font-size: 4vh;
}

#scrollProgress {
  position: fixed;
  bottom: 10px;
  left: 10px;
  z-index: 99;
  font-size: 3vh;
}
</style>
