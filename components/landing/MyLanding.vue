<template>
  <body>
    <div class="flex h-screen bg-black">
      <div class="w-1/2 flex flex-col justify-center px-8">
        <div class="core mx-auto">
          <h1 class="text-white text-6xl mb-8 font-exo leading-none">
            <span class="text-blue-400">AMINE SANHAJI</span>, PORTFOLIO EFFETS
            SYMPA THREEJS
          </h1>
          <p class="text-gray-400 mb-8 font-exa text-2xl">
            Dans le cadre d'une montée en compétence en Threejs, voici quelques
            effets et modèles réalisées avec ce framwork, ainsi que les
            Vertex/Fragment shaders. <br />
            (niveau débutant)
          </p>
          <div>
            <a
              href="/home"
              class="text-white bg-green-600 inline-block px-8 py-4 rounded-full"
              >Par ici</a
            >
          </div>
        </div>
      </div>
      <div class="w-1/2" id="canvasContainer">
        <canvas></canvas>
      </div>
    </div>
    <script type="x-shader/x-vertex" id="vertexshader">
      varying vec2 vertexUV;
      varying vec3 vertexNormal;

        void main(){
          vertexUV = uv;
          vertexNormal = normalize(normalMatrix * normal);

        gl_Position= projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    </script>

    <script type="x-shader/x-fragment" id="fragmentshader">
      uniform sampler2D globeTexture;

      varying vec2 vertexUV;
      varying vec3 vertexNormal;

        void main() {
          float intensity = 1.05 - dot(
            vertexNormal, vec3(0.0, 0.0, 1.0));
          vec3 atmosphere = vec3(0.3, 0.6, 1.0) *
            pow(intensity, 1.5);

          gl_FragColor = vec4(atmosphere + texture2D(globeTexture, vertexUV).xyz, 1.0);
        }
    </script>
    <!-- --------ATMOSPHERE SHADERS ---------- -->

    <script type="x-shader/x-vertex" id="atmosphereVertex">
      varying vec3 vertexNormal;

        void main(){
          vertexNormal = normalize(normalMatrix * normal);
        gl_Position= projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    </script>

    <script type="x-shader/x-fragment" id="atmosphereFragment">
      varying vec3 vertexNormal;

        void main() {
          float intensity = pow(0.9 - dot(vertexNormal, vec3(0, 0, 1.0)), 2.0);
          gl_FragColor = vec4(0.3, 0.6, 1.0, 1.0) * intensity;
        }
    </script>
  </body>
</template>

<script>
import gsap from "gsap";
import * as THREE from "three";
// import vertexShader from "@/components/shaders/vertex.glsl";
export default {
  mounted() {
    const canvasContainer = document.querySelector("#canvasContainer");

    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      canvasContainer.offsetWidth / canvasContainer.offsetHeight,
      0.1,
      1000
    );

    const renderer = new THREE.WebGLRenderer({
      antialias: true,
      canvas: document.querySelector("canvas"),
    });

    renderer.setSize(canvasContainer.offsetWidth, canvasContainer.offsetHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    // document.body.appendChild(renderer.domElement); generate canvas

    // texture
    const loader = new THREE.TextureLoader();
    const globe = loader.load(require("@/assets/globe2.jpg"));

    //create sphere
    const sphere = new THREE.Mesh(
      new THREE.SphereGeometry(5, 50, 50),
      new THREE.ShaderMaterial({
        vertexShader: document.getElementById("vertexshader").textContent,
        fragmentShader: document.getElementById("fragmentshader").textContent,
        uniforms: {
          globeTexture: { value: globe },
        },
      })
    );

    // scene.add(sphere);

    //create atmosphere
    const atmosphere = new THREE.Mesh(
      new THREE.SphereGeometry(5, 50, 50),
      new THREE.ShaderMaterial({
        vertexShader: document.getElementById("atmosphereVertex").textContent,
        fragmentShader:
          document.getElementById("atmosphereFragment").textContent,
        blending: THREE.AdditiveBlending,
        side: THREE.BackSide,
      })
    );
    atmosphere.scale.set(1.1, 1.1, 1.1);

    scene.add(atmosphere);

    const group = new THREE.Group();
    group.add(sphere);
    scene.add(group);

    const starGeometry = new THREE.BufferGeometry();
    const starMaterial = new THREE.PointsMaterial({
      color: 0xffffff,
    });

    const starVertices = [];
    for (let i = 0; i < 10000; i++) {
      const x = (Math.random() - 0.5) * 2000;
      const y = (Math.random() - 0.5) * 2000;
      const z = -Math.random() * 3000;
      starVertices.push(x, y, z);
    }

    starGeometry.setAttribute(
      "position",
      new THREE.Float32BufferAttribute(starVertices, 3)
    );

    const stars = new THREE.Points(starGeometry, starMaterial);
    scene.add(stars);

    camera.position.z = 15;

    const mouse = {
      x: undefined,
      y: undefined,
    };

    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
      sphere.rotation.y += 0.002;
      // group.rotation.y = mouse.x * 0.5;
      gsap.to(group.rotation, {
        x: -mouse.y * 0.3,
        y: mouse.x * 0.5,
        duration: 2,
      });
    }
    animate();

    addEventListener("mousemove", () => {
      mouse.x = (event.clientX / innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / innerHeight) * 2 + 1;
    });
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Hubballi&family=Lato&display=swap");
body {
  margin: 0;
  -webkit-font-smoothing: antialiased;
}
.font-exo {
  font-family: "Hubballi", cursive;
}
.font-exa {
  font-family: "Lato", sans-serif;
}
.core {
  max-width: 500px;
}
</style>
