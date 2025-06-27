<script>
import { ref, onMounted } from "vue";
import * as BABYLON from "@babylonjs/core";

export default {
  setup() {
    const canvas = ref(null);
    const distance = ref(0);
    const planetSize = ref(100);

    onMounted(() => {
      const engine = new BABYLON.Engine(canvas.value, true);
      const scene = new BABYLON.Scene(engine);
      scene.clearColor = new BABYLON.Color3(0.02, 0.02, 0.1); // Тёмный фон

      const camera = new BABYLON.ArcRotateCamera(
        "camera",
        Math.PI / 2,
        Math.PI / 4,
        10,
        BABYLON.Vector3.Zero(),
        scene
      );
      camera.attachControl(canvas.value, true);

      const light = new BABYLON.HemisphericLight(
        "light",
        new BABYLON.Vector3(0, 1, 0),
        scene
      );
      light.intensity = 0.7;

      for (let i = 0; i < 500; i++) {
        const star = BABYLON.MeshBuilder.CreateSphere(
          `star-${i}`,
          {
            diameter: 0.05 + Math.random() * 0.1,
            segments: 4,
          },
          scene
        );

        star.position = new BABYLON.Vector3(
          (Math.random() - 0.5) * 100,
          (Math.random() - 0.5) * 100,
          (Math.random() - 0.5) * 100
        );

        const starMaterial = new BABYLON.StandardMaterial(
          `starMat-${i}`,
          scene
        );
        starMaterial.emissiveColor = new BABYLON.Color3(1, 1, 1); // Белый цвет
        star.material = starMaterial;
      }

      const planet = BABYLON.MeshBuilder.CreateSphere(
        "planet",
        {
          diameter: 2,
          segments: 32,
        },
        scene
      );

      const planetMaterial = new BABYLON.StandardMaterial(
        "planetMaterial",
        scene
      );
      planetMaterial.diffuseColor = new BABYLON.Color3(0.3, 0.5, 0.9); // Синий цвет
      planet.material = planetMaterial;

      scene.registerBeforeRender(() => {
        // Вращение по X и Y
        planet.rotation.x += 0.005;
        planet.rotation.y += 0.01;

        distance.value = BABYLON.Vector3.Distance(
          camera.position,
          planet.position
        );

        const maxDistance = 20;
        const scale = Math.max(0.1, 1 - distance.value / maxDistance);
        planetSize.value = scale * 100;
        planet.scaling.setAll(scale);
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

<template>
  <div class="space">
    <canvas ref="canvas"></canvas>
    <div class="info">
      Расстояние: {{ Math.round(distance) }} | Размер:
      {{ Math.round(planetSize) }}%
    </div>
  </div>
</template>

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
