<template>
  <body>
    <div class="flex h-screen bg-black">
      <div class="w-1/2" id="magic">
      </div>
      <div class="playground">
      <div class="bottomPosition">
        <a href="/home" class="text-white bg-green-600 inline-block px-8 py-4 rounded-full">Par ici</a>
      </div>
    </div>
      <div class="w-1/2 relative" id="canvasContainer">
        <div class="container flex items-center justify-center absolute z-10 w-full h-screen">
          <div class="content">
            <h1>DEVELOPPEUR WEB </h1>
            <!-- <p>Ici le coeur est grand comme l'égo <br />"Napo"</p> -->
          </div>
        </div>
        <canvas id="Globy"></canvas>
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

    <!-- --------TEXT SHADERS ---------- -->

     <script type="x-shader/x-vertex" id="textVertex">

      attribute float size;
      attribute vec3 customColor;
      varying vec3 vColor;

      void main() {

        vColor = customColor;
        vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
        gl_PointSize = size * ( 300.0 / -mvPosition.z );
        gl_Position = projectionMatrix * mvPosition;

      }
    </script>
    <script type="x-shader/x-fragment" id="textFragment">

      uniform vec3 color;
      uniform sampler2D pointTexture;

      varying vec3 vColor;

      void main() {

        gl_FragColor = vec4( color * vColor, 1.0 );
        gl_FragColor = gl_FragColor * texture2D( pointTexture, gl_PointCoord );

      }
    </script>
  </body>
</template>

<script>
import gsap from "gsap";
import * as THREE from "three";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader.js";

import CSSRulePlugin from "gsap/CSSRulePlugin";
gsap.registerPlugin(CSSRulePlugin);
export default {
  mounted() {
    this.GlobeLand()
    this.CssPlugin();
    this.TextLand()
  },
  methods: {
    GlobeLand() {
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
      canvas: document.querySelector("#Globy"),
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
    CssPlugin() {
      const content = CSSRulePlugin.getRule(".content:before");
      const h1 = document.querySelector("h1");
      const p = document.querySelector("p");
      const tl = gsap.timeline();

      tl.from(content, { delay: 0.5, duration: 4, cssRule: { scaleX: 0 } });
      tl.to(
        h1,
        {
          duration: 2,
          clipPath: "polygon(0 0, 100% 0, 100% 100%, 0 100%)",
          y: "30px",
        },
        "-=3"
      );
      tl.to(
        p,
        {
          duration: 4,
          clipPath: "polygon(0 0, 100% 0, 100% 100%, 0 100%)",
          y: "30px",
        },
        "-=2"
      );
    },
    TextLand() {
      const preload = () => {
      const manager = new THREE.LoadingManager();
      manager.onLoad = function () {
        const environment = new Environment(typo, particle);
      };

      var typo = null;
      const loader = new FontLoader(manager);
      const font = loader.load(
        "https://res.cloudinary.com/dydre7amr/raw/upload/v1612950355/font_zsd4dr.json",
        function (font) {
          typo = font;
        }
      );
      const particle = new THREE.TextureLoader(manager).load(
        "https://res.cloudinary.com/dfvtkoboz/image/upload/v1605013866/particle_a64uzf.png"
      );
    };

    if (
      document.readyState === "complete" ||
      (document.readyState !== "loading" && !document.documentElement.doScroll)
    )
      preload();
    else document.addEventListener("DOMContentLoaded", preload);

    class Environment {
      constructor(font, particle) {
        this.font = font;
        this.particle = particle;
        this.container = document.querySelector("#magic"); 
        this.scene = new THREE.Scene();
        this.createCamera();
        this.createRenderer();
        this.setup();
        this.bindEvents();
      }

      bindEvents() {
        window.addEventListener("resize", this.onWindowResize.bind(this));
      }

      setup() {
        this.createParticles = new CreateParticles(
          this.scene,
          this.font,
          this.particle,
          this.camera,
          this.renderer
        );
      }

      render() {
        this.createParticles.render();
        this.renderer.render(this.scene, this.camera);
      }

      createCamera() {
        this.camera = new THREE.PerspectiveCamera(
          65,
          this.container.clientWidth / this.container.clientHeight,
          1,
          10000
        );
        this.camera.position.set(0, 0, 100);
      }

      createRenderer() {
        this.renderer = new THREE.WebGLRenderer();
        this.renderer.setSize(
          this.container.offsetWidth ,
          this.container.offsetHeight 
        );

        this.renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        this.renderer.outputEncoding = THREE.sRGBEncoding;
        this.container.appendChild(this.renderer.domElement);

        this.renderer.setAnimationLoop(() => {
          this.render();
        });
      }

      onWindowResize() {
        this.camera.aspect =
          this.container.clientWidth / this.container.clientHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(
          this.container.clientWidth,
          this.container.clientHeight
        );
      }
    }

    class CreateParticles {
      constructor(scene, font, particleImg, camera, renderer) {
        this.scene = scene;
        this.font = font;
        this.particleImg = particleImg;
        this.camera = camera;
        this.renderer = renderer;

        this.raycaster = new THREE.Raycaster();
        this.mouse = new THREE.Vector2(-200, 200);

        this.colorChange = new THREE.Color();

        this.buttom = false;

        this.data = {
          text: "AMINE\nSANHAJI",
          amount: 1500,
          particleSize: 1,
          particleColor: 0xffffff,
          textSize: 14,
          area: 250,
          ease: 0.05,
        };

        this.setup();
        this.bindEvents();
      }

      setup() {
        const geometry = new THREE.PlaneGeometry(
          this.visibleWidthAtZDepth(100, this.camera),
          this.visibleHeightAtZDepth(100, this.camera)
        );
        const material = new THREE.MeshBasicMaterial({
          color: 0x00ff00,
          transparent: true,
        });
        this.planeArea = new THREE.Mesh(geometry, material);
        this.planeArea.visible = false;
        this.createText();
      }

      bindEvents() {
        document.addEventListener("mousedown", this.onMouseDown.bind(this));
        document.addEventListener("mousemove", this.onMouseMove.bind(this));
        document.addEventListener("mouseup", this.onMouseUp.bind(this));
      }

      onMouseDown() {
        this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

        const vector = new THREE.Vector3(this.mouse.x, this.mouse.y, 0.5);
        vector.unproject(this.camera);
        const dir = vector.sub(this.camera.position).normalize();
        const distance = -this.camera.position.z / dir.z;
        this.currenPosition = this.camera.position
          .clone()
          .add(dir.multiplyScalar(distance));

        const pos = this.particles.geometry.attributes.position;
        this.buttom = true;
        this.data.ease = 0.01;
      }

      onMouseUp() {
        this.buttom = false;
        this.data.ease = 0.05;
      }

      onMouseMove() {
        this.mouse.x = (event.clientX / window.innerWidth) * 4 - 1;
        this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      }

      render(level) {
        const time = ((0.001 * performance.now()) % 12) / 12;
        const zigzagTime = (1 + Math.sin(time * 2 * Math.PI)) / 6;

        this.raycaster.setFromCamera(this.mouse, this.camera);

        const intersects = this.raycaster.intersectObject(this.planeArea);

        if (intersects.length > 0) {
          const pos = this.particles.geometry.attributes.position;
          const copy = this.geometryCopy.attributes.position;
          const coulors = this.particles.geometry.attributes.customColor;
          const size = this.particles.geometry.attributes.size;

          const mx = intersects[0].point.x;
          const my = intersects[0].point.y;
          const mz = intersects[0].point.z;

          for (var i = 0, l = pos.count; i < l; i++) {
            const initX = copy.getX(i);
            const initY = copy.getY(i);
            const initZ = copy.getZ(i);

            let px = pos.getX(i);
            let py = pos.getY(i);
            let pz = pos.getZ(i);

            this.colorChange.setHSL(0.5, 1, 1);
            coulors.setXYZ(
              i,
              this.colorChange.r,
              this.colorChange.g,
              this.colorChange.b
            );
            coulors.needsUpdate = true;

            size.array[i] = this.data.particleSize;
            size.needsUpdate = true;

            let dx = mx - px;
            let dy = my - py;
            const dz = mz - pz;

            const mouseDistance = this.distance(mx, my, px, py);
            let d = (dx = mx - px) * dx + (dy = my - py) * dy;
            const f = -this.data.area / d;

            if (this.buttom) {
              const t = Math.atan2(dy, dx);
              px -= f * Math.cos(t);
              py -= f * Math.sin(t);

              this.colorChange.setHSL(0.5 + zigzagTime, 1.0, 0.5);
              coulors.setXYZ(
                i,
                this.colorChange.r,
                this.colorChange.g,
                this.colorChange.b
              );
              coulors.needsUpdate = true;

              if (
                px > initX + 70 ||
                px < initX - 70 ||
                py > initY + 70 ||
                py < initY - 70
              ) {
                this.colorChange.setHSL(0.15, 1.0, 0.5);
                coulors.setXYZ(
                  i,
                  this.colorChange.r,
                  this.colorChange.g,
                  this.colorChange.b
                );
                coulors.needsUpdate = true;
              }
            } else {
              if (mouseDistance < this.data.area) {
                if (i % 5 == 0) {
                  const t = Math.atan2(dy, dx);
                  px -= 0.03 * Math.cos(t);
                  py -= 0.03 * Math.sin(t);

                  this.colorChange.setHSL(0.15, 1.0, 0.5);
                  coulors.setXYZ(
                    i,
                    this.colorChange.r,
                    this.colorChange.g,
                    this.colorChange.b
                  );
                  coulors.needsUpdate = true;

                  size.array[i] = this.data.particleSize / 1.2;
                  size.needsUpdate = true;
                } else {
                  const t = Math.atan2(dy, dx);
                  px += f * Math.cos(t);
                  py += f * Math.sin(t);

                  pos.setXYZ(i, px, py, pz);
                  pos.needsUpdate = true;

                  size.array[i] = this.data.particleSize * 1.3;
                  size.needsUpdate = true;
                }

                if (
                  px > initX + 10 ||
                  px < initX - 10 ||
                  py > initY + 10 ||
                  py < initY - 10
                ) {
                  this.colorChange.setHSL(0.15, 1.0, 0.5);
                  coulors.setXYZ(
                    i,
                    this.colorChange.r,
                    this.colorChange.g,
                    this.colorChange.b
                  );
                  coulors.needsUpdate = true;

                  size.array[i] = this.data.particleSize / 1.8;
                  size.needsUpdate = true;
                }
              }
            }

            px += (initX - px) * this.data.ease;
            py += (initY - py) * this.data.ease;
            pz += (initZ - pz) * this.data.ease;

            pos.setXYZ(i, px, py, pz);
            pos.needsUpdate = true;
          }
        }
      }

      createText() {
        let thePoints = [];

        let shapes = this.font.generateShapes(
          this.data.text,
          this.data.textSize
        );
        let geometry = new THREE.ShapeGeometry(shapes);
        geometry.computeBoundingBox();

        const xMid =
          -0.5 * (geometry.boundingBox.max.x - geometry.boundingBox.min.x);
        const yMid =
          (geometry.boundingBox.max.y - geometry.boundingBox.min.y) / 2.85;

        geometry.center();

        let holeShapes = [];

        for (let q = 0; q < shapes.length; q++) {
          let shape = shapes[q];

          if (shape.holes && shape.holes.length > 0) {
            for (let j = 0; j < shape.holes.length; j++) {
              let hole = shape.holes[j];
              holeShapes.push(hole);
            }
          }
        }
        shapes.push.apply(shapes, holeShapes);

        let colors = [];
        let sizes = [];

        for (let x = 0; x < shapes.length; x++) {
          let shape = shapes[x];

          const amountPoints =
            shape.type == "Path" ? this.data.amount / 2 : this.data.amount;

          let points = shape.getSpacedPoints(amountPoints);

          points.forEach((element, z) => {
            const a = new THREE.Vector3(element.x, element.y, 0);
            thePoints.push(a);
            colors.push(
              this.colorChange.r,
              this.colorChange.g,
              this.colorChange.b
            );
            sizes.push(1);
          });
        }

        let geoParticles = new THREE.BufferGeometry().setFromPoints(thePoints);
        geoParticles.translate(xMid, yMid, 0);

        geoParticles.setAttribute(
          "customColor",
          new THREE.Float32BufferAttribute(colors, 3)
        );

        geoParticles.setAttribute(
          "size",
          new THREE.Float32BufferAttribute(sizes, 1)
        );

    
        const material = new THREE.ShaderMaterial({
          uniforms: {
            color: { value: new THREE.Color(0xffffff) },
            pointTexture: { value: this.particleImg },
          },

          vertexShader: document.getElementById("textVertex").textContent,
          fragmentShader: document.getElementById("textFragment").textContent,

          blending: THREE.AdditiveBlending,
          depthTest: false,
          transparent: true,
        });
        
        this.particles = new THREE.Points(geoParticles, material);
        this.scene.add(this.particles);

        this.geometryCopy = new THREE.BufferGeometry();
        this.geometryCopy.copy(this.particles.geometry);
      }

      visibleHeightAtZDepth(depth, camera) {
        const cameraOffset = camera.position.z;
        if (depth < cameraOffset) depth -= cameraOffset;
        else depth += cameraOffset;

        const vFOV = (camera.fov * Math.PI) / 180;

        return 2 * Math.tan(vFOV / 2) * Math.abs(depth);
      }

      visibleWidthAtZDepth(depth, camera) {
        const height = this.visibleHeightAtZDepth(depth, camera);
        return height * camera.aspect;
      }

      distance(x1, y1, x2, y2) {
        return Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
      }
    }
    },
  }
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Hubballi&family=Lato&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Roboto&display=swap");
@import url("https://res.cloudinary.com/dydre7amr/raw/upload/v1612950355/font_zsd4dr.json");
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
#Texty {
  position: fixed;
  width: 100%;
  height: 100vh;
  display: block;
  top: 0;
  left: 0;
  z-index: -9999;
}
.playground {
  position: fixed;
  width: 100%;
  height: 100vh;
  display: block;
  top: 0;
  left: 8rem;
  display: flex;
  flex-wrap: nowrap;
  flex-direction: column;
  justify-content: flex-end;
  align-items: center;
}
.bottomPosition {
  text-align: center;
  margin-bottom: 50px;
  align-self: flex-start;
}
.minText {
  font-size: 14px;
}

a {
  color: white;
  font-size: 23px;
  text-decoration: none;
}
/* .container {
  position: absolute;
  z-index: 1;
  width: 100%;
  height: 100vh;
  display: flex;
  place-content: center;
} */
.content {
  display: flex;
  gap: 5em;
  width: 100%;
  padding-top: 3em;
  position: relative;
  color: bisque;
}

.content:before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  border-bottom: 1px solid white;
  transform: scaleX(1);
}

h1 {
  font-size: 4em;
  /* width: 50vw; */
  line-height: 97%;
  text-align: center;
}

h1,
p {
  flex-basis: 0;
  flex-grow: 1;
  clip-path: polygon(0 0, 100% 0, 100% 0, 0 0);
}

p {
  font-size: 1.3rem;
  width: 40vw;
}
</style>
