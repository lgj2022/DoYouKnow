<template>
  <div class="cavas-container">
    <!-- <div class="moon-background"></div>
    <div class="sun-background"></div> -->
  </div>
</template>

<script>
import {
  DoubleSide,
  PCFSoftShadowMap,
  MeshPhysicalMaterial,
  TextureLoader,
  FloatType,
  PMREMGenerator,
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  Color,
  ACESFilmicToneMapping,
  sRGBEncoding,
  Mesh,
  SphereGeometry,
  MeshBasicMaterial,
  Vector2,
  DirectionalLight,
  Clock,
  RingGeometry,
  Vector3,
  PlaneGeometry,
  Group,
} from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader";

export default {
  setup() {
    const scene = new Scene();

    // let sunBackground = document.querySelector(".sun-background");
    // let moonBackground = document.querySelector(".moon-background");

    const ringsScene = new Scene();

    const camera = new PerspectiveCamera(
      45,
      innerWidth / innerHeight,
      0.1,
      1000
    );
    camera.position.set(0, 15, 50);

    const ringsCamera = new PerspectiveCamera(
      45,
      innerWidth / innerHeight,
      0.1,
      1000
    );
    ringsCamera.position.set(0, 0, 50);

    const renderer = new WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(innerWidth, innerHeight);
    renderer.toneMapping = ACESFilmicToneMapping;
    renderer.outputEncoding = sRGBEncoding;
    renderer.physicallyCorrectLights = true;
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = PCFSoftShadowMap;
    document.body.appendChild(renderer.domElement);

    const sunLight = new DirectionalLight(
      new Color("#FFFFFF").convertSRGBToLinear(),
      3.5
    );
    sunLight.position.set(10, 20, 10);
    sunLight.castShadow = true;
    sunLight.shadow.mapSize.width = 512;
    sunLight.shadow.mapSize.height = 512;
    sunLight.shadow.camera.near = 0.5;
    sunLight.shadow.camera.far = 100;
    sunLight.shadow.camera.left = -10;
    sunLight.shadow.camera.bottom = -10;
    sunLight.shadow.camera.top = 10;
    sunLight.shadow.camera.right = 10;
    scene.add(sunLight);

    const moonLight = new DirectionalLight(
      new Color("#77ccff").convertSRGBToLinear(),
      0
    );
    moonLight.position.set(-10, 20, 10);
    moonLight.castShadow = true;
    moonLight.shadow.mapSize.width = 512;
    moonLight.shadow.mapSize.height = 512;
    moonLight.shadow.camera.near = 0.5;
    moonLight.shadow.camera.far = 100;
    moonLight.shadow.camera.left = -10;
    moonLight.shadow.camera.bottom = -10;
    moonLight.shadow.camera.top = 10;
    moonLight.shadow.camera.right = 10;
    scene.add(moonLight);

    // orbitcontrols
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);
    controls.dampingFactor = 0.05;
    controls.enableDamping = true;
    controls.enableZoom = false;

    let mousePos = new Vector2(0, 0);

    (async function () {
      let pmrem = new PMREMGenerator(renderer);
      let envmapTexture = await new RGBELoader()
        .setDataType(FloatType)
        .loadAsync("texture/4k.hdr");

      //globetexture
      let envMap = pmrem.fromEquirectangular(envmapTexture).texture;
      let textures = {
        bump: await new TextureLoader().loadAsync("texture/earthbump.jpg"),
        map: await new TextureLoader().loadAsync("texture/earthsample4.jpg"),
        spec: await new TextureLoader().loadAsync("texture/earthspec.jpg"),
        planeTrailMask: await new TextureLoader().loadAsync("texture/mask.png"),
      };

      //globe
      let sphere = new Mesh(
        new SphereGeometry(11.5, 70, 70),
        new MeshPhysicalMaterial({
          map: textures.map,
          roughnessMap: textures.spec,
          bumpMap: textures.bump,
          bumpScale: 0.05,
          envMap,
          envMapIntensity: 0.4,
          sheen: 1,
          sheenRoughness: 0.75,
          sheenColor: new Color("#ff8a00").convertSRGBToLinear(),
          clearcoat: 0.5,
        })
      );
      console.log(sphere, "=========");
      sphere.sunEnvIntensity = 0.4;
      sphere.moonEnvIntensity = 0.1;
      sphere.rotation.y += Math.PI * 1.25;
      sphere.receiveShadow = true;
      scene.add(sphere);

      const ring1 = new Mesh(
        new RingGeometry(15, 13.5, 80, 1, 0),
        new MeshPhysicalMaterial({
          color: new Color("#FFCB8E").convertSRGBToLinear().multiplyScalar(200),
          roughness: 0.25,
          envMap,
          envMapIntensity: 1.8,
          side: DoubleSide,
          transparent: true,
          opacity: 0.35,
        })
      );
      ring1.name = "ring";
      ring1.sunOpacity = 0.35;
      ring1.moonOpacity = 0.03;
      ringsScene.add(ring1);

      const ring2 = new Mesh(
        new RingGeometry(16.5, 15.75, 80, 1, 0),
        new MeshBasicMaterial({
          color: new Color("#FFCB8E").convertSRGBToLinear(),
          transparent: true,
          opacity: 0.5,
          side: DoubleSide,
        })
      );
      ring2.name = "ring";
      ring2.sunOpacity = 0.35;
      ring2.moonOpacity = 0.1;
      ringsScene.add(ring2);

      const ring3 = new Mesh(
        new RingGeometry(18, 17.75, 80),
        new MeshBasicMaterial({
          color: new Color("#FFCB8E").convertSRGBToLinear().multiplyScalar(50),
          transparent: true,
          opacity: 0.5,
          side: DoubleSide,
        })
      );
      ring3.name = "ring";
      ring3.sunOpacity = 0.35;
      ring3.moonOpacity = 0.03;
      ringsScene.add(ring3);

      let plane = (await new GLTFLoader().loadAsync("texture/scene.glb")).scene
        .children[0];

      let planesData = [
        makePlane(plane, textures.planeTrailMask, envMap, scene),
        makePlane(plane, textures.planeTrailMask, envMap, scene),
        makePlane(plane, textures.planeTrailMask, envMap, scene),
        makePlane(plane, textures.planeTrailMask, envMap, scene),
        makePlane(plane, textures.planeTrailMask, envMap, scene),
      ];

      // let daytime = true;
      // let animating = false;
      // window.addEventListener("mousemove", (e) => {
      //   if (animating) return;

      //   let anim = [0, 1];

      //   if (e.clientX > innerWidth - 200 && !daytime) {
      //     anim = [1, 0];
      //   } else if (e.clientX < 200 && daytime) {
      //     anim = [0, 1];
      //   } else {
      //     return;
      //   }

      //   animating = true;

      //   let obj = { t: 0 };
      //   anime({
      //     targets: obj,
      //     t: anim,
      //     complete: () => {
      //       animating = false;
      //       daytime = !daytime;
      //     },
      //     update: () => {
      //       sunLight.intensity = 3.5 * (1 - obj.t);
      //       moonLight.intensity = 3.5 * obj.t;

      //       sunLight.position.setY(20 * (1 - obj.t));
      //       moonLight.position.setY(20 * obj.t);

      //       sphere.material.sheen = 1 - obj.t;
      //       scene.children.forEach((child) => {
      //         child.traverse((object) => {
      //           if (object instanceof Mesh && object.material.envMap) {
      //             object.material.envMapIntensity =
      //               object.sunEnvIntensity * (1 - obj.t) +
      //               object.moonEnvIntensity * obj.t;
      //           }
      //         });
      //       });

      //       ringsScene.children.forEach((child, i) => {
      //         child.traverse((object) => {
      //           object.material.opacity =
      //             object.sunOpacity * (1 - obj.t) + object.moonOpacity * obj.t;
      //         });
      //       });

      //       sunBackground.style.opacity = 1 - obj.t;
      //       moonBackground.style.opacity = obj.t;
      //     },
      //     easing: "easeInOutSine",
      //     duration: 500,
      //   });
      // });

      let clock = new Clock();

      renderer.setAnimationLoop(() => {
        let delta = clock.getDelta();
        // scene.rotation.y += delta * 0.05;

        controls.update();
        renderer.render(scene, camera);

        ring1.rotation.x = ring1.rotation.x * 0.95 + mousePos.y * 0.05 * 1.2;
        ring1.rotation.y = ring1.rotation.y * 0.95 + mousePos.x * 0.05 * 1.2;

        ring2.rotation.x = ring2.rotation.x * 0.95 + mousePos.y * 0.05 * 0.375;
        ring2.rotation.y = ring2.rotation.y * 0.95 + mousePos.x * 0.05 * 0.375;

        ring3.rotation.x = ring3.rotation.x * 0.95 - mousePos.y * 0.05 * 0.275;
        ring3.rotation.y = ring3.rotation.y * 0.95 - mousePos.x * 0.05 * 0.275;

        planesData.forEach((planeData) => {
          let plane = planeData.group;

          plane.position.set(0, 0, 0);
          plane.rotation.set(0, 0, 0);
          plane.updateMatrixWorld();

          planeData.rot += delta;
          plane.rotateOnAxis(planeData.randomAxis, planeData.randomAxisRot); // random axis
          plane.rotateOnAxis(new Vector3(0, 1, 0), planeData.rot); // y-axis rotation
          plane.rotateOnAxis(new Vector3(0, 0, 1.3), planeData.rad); // this decides the radius
          plane.translateY(planeData.yOff);
          plane.rotateOnAxis(new Vector3(1, 0, 0), +Math.PI * 0.5);
        });

        renderer.autoClear = false;
        renderer.render(ringsScene, ringsCamera);
        renderer.autoClear = true;
      });
    })();

    function nr() {
      return Math.random() * 2 - 1;
    }

    function makePlane(planeMesh, trailTexture, envMap, scene) {
      let plane = planeMesh.clone();
      plane.scale.set(0.001, 0.001, 0.001);
      plane.position.set(0, 0, 0);
      plane.rotation.set(0, 0, 0);
      plane.updateMatrixWorld();

      plane.traverse((object) => {
        if (object instanceof Mesh) {
          object.material.envMap = envMap;
          object.sunEnvIntensity = 1;
          object.moonEnvIntensity = 0.3;
          object.castShadow = true;
          object.receiveShadow = true;
        }
      });

      let trail = new Mesh(
        new PlaneGeometry(1, 2),
        new MeshPhysicalMaterial({
          envMap,
          envMapIntensity: 3,

          roughness: 0.4,
          metalness: 0,
          transmission: 1,

          transparent: true,
          opacity: 1,
          alphaMap: trailTexture,
        })
      );
      trail.sunEnvIntensity = 3;
      trail.moonEnvIntensity = 0.7;
      trail.rotateX(Math.PI);
      trail.translateY(1.1);

      let group = new Group();
      group.add(plane);
      group.add(trail);

      scene.add(group);

      return {
        group,
        yOff: 10.5 + Math.random() * 1.0,
        rot: Math.PI * 2, // just to set a random starting point
        rad: Math.random() * Math.PI * 0.45 + Math.PI * 0.05,
        randomAxis: new Vector3(nr(), nr(), nr()).normalize(),
        randomAxisRot: Math.random() * Math.PI * 2,
      };
    }

    // pointer Load
    function convertLatLng(p) {
      let lat = (90 - p.lat * Math.PI) / 180;
      let lng = (180 + p.lng * Math.PI) / 180;

      let x = -Math.sin(lat) * Math.cos(lng) - 3;
      let y = Math.sin(lat) * Math.sin(lng) + 8;
      let z = Math.cos(lat) + 10;

      return {
        x,
        y,
        z,
      };
    }

    // country coordinates
    let point1 = {
      lat: 38.895,
      lng: 77.015,
    };
    let pos = convertLatLng(point1);

    let pointer = new GLTFLoader();
    pointer.load("texture/pointer.glb", function (gltf) {
      gltf.scene.position.set(pos.x, pos.y, pos.z);
      gltf.scene.scale.set(0.05, 0.05, 0.05);
      scene.add(gltf.scene);
    });
    // pointer.load("texture/pointer.glb", function (gltf) {
    //   gltf.scene.position.set(pos.x - 3.5, pos.y, pos.z - 19.5);
    //   gltf.scene.scale.set(0.05, 0.05, 0.05);
    //   scene.add(gltf.scene);
    // });
    // pointer.load("texture/pointer.glb", function (gltf) {
    //   gltf.scene.position.set(pos.x + 11, pos.y + 2.5, pos.z - 8);
    //   gltf.scene.scale.set(0.05, 0.05, 0.05);
    //   scene.add(gltf.scene);
    // });

    window.addEventListener("mousemove", (e) => {
      let x = e.clientX - innerWidth * 0.5;
      let y = e.clientY - innerHeight * 0.5;

      mousePos.x = x * 0.0003;
      mousePos.y = y * 0.0003;
    });
  },
};
</script>

<style>
.sun-background {
  position: absolute;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, rgb(255 219 158), rgb(253 243 220));
  opacity: 1;
}
.moon-background {
  position: absolute;
  width: 100%;
  height: 100%;
  background: linear-gradient(313deg, #0b1a2b, #3a6291 111%);
  opacity: 1;
}
canvas {
  position: relative;
}
</style>
