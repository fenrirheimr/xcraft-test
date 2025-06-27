<template>
  <div class="space">
    <canvas ref="canvas"></canvas>
    <div class="info">
      Расстояние: {{ Math.round(distance) }} | Размер:
      {{ Math.round(planetSize) }}%
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import * as BABYLON from "@babylonjs/core";

export default {
  setup() {
    const canvas = ref(null);
    const distance = ref(0);
    const planetSize = ref(100);
    const planetDiameter = 2;
    const maxDistance = planetDiameter * 10;

    onMounted(() => {
      const engine = new BABYLON.Engine(canvas.value, true);
      const scene = new BABYLON.Scene(engine);
      scene.clearColor = new BABYLON.Color4(0, 0, 0.1, 1);

      const camera = new BABYLON.ArcRotateCamera(
        "camera",
        Math.PI / 2,
        Math.PI / 4,
        10,
        BABYLON.Vector3.Zero(),
        scene
      );
      camera.attachControl(canvas.value, true);
      camera.lowerRadiusLimit = planetDiameter * 0.5;
      camera.upperRadiusLimit = maxDistance;

      const sun = new BABYLON.DirectionalLight(
        "sun",
        new BABYLON.Vector3(-1, -1, -1),
        scene
      );
      sun.intensity = 0.8;

      for (let i = 0; i < 500; i++) {
        const star = BABYLON.MeshBuilder.CreateSphere(
          `star`,
          { diameter: 0.05 + Math.random() * 0.1 },
          scene
        );
        star.position = new BABYLON.Vector3(
          (Math.random() - 0.5) * 200,
          (Math.random() - 0.5) * 200,
          (Math.random() - 0.5) * 200
        );
        const starColor = new BABYLON.StandardMaterial("starColor", scene);
        starColor.emissiveColor = new BABYLON.Color3(1, 1, 1);
        star.material = starColor;
      }

      const planet = BABYLON.MeshBuilder.CreateSphere(
        "planet",
        { diameter: planetDiameter, segments: 32 },
        scene
      );
      const planetColor = new BABYLON.StandardMaterial("planetColor", scene);
      planetColor.diffuseColor = new BABYLON.Color3(0.3, 0.5, 0.9);
      planet.material = planetColor;

      scene.registerBeforeRender(() => {
        planet.rotation.x += 0.005;
        planet.rotation.y += 0.01;

        distance.value = BABYLON.Vector3.Distance(
          camera.position,
          planet.position
        );

        if (distance.value >= maxDistance) {
          planet.setEnabled(false);
          planetSize.value = 0;
        } else {
          planet.setEnabled(true);
          const scale = 1 - distance.value / maxDistance;
          planetSize.value = scale * 100;
          planet.scaling.setAll(scale);
        }
      });

      engine.runRenderLoop(() => {
        scene.render();
      });

      window.addEventListener("resize", () => {
        engine.resize();
      });
    });

    return { canvas, distance, planetSize };
  },
};
</script>

<style scoped>
.space {
  position: relative;
  width: 100%;
  height: 100vh;
}
.space canvas {
  width: 100%;
  height: 100%;
  display: block;
}
.info {
  position: absolute;
  top: 20px;
  left: 20px;
  color: white;
  background: rgba(0, 0, 0, 0.7);
  padding: 10px;
  border-radius: 5px;
  font-family: Arial;
}
</style>
