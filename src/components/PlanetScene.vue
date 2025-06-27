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

    onMounted(() => {
      // 1. Настраиваем сцену
      const engine = new BABYLON.Engine(canvas.value, true);
      const scene = new BABYLON.Scene(engine);
      scene.clearColor = new BABYLON.Color3(0.02, 0.02, 0.1); // Тёмный фон

      // 2. Добавляем камеру (как наши глаза)
      const camera = new BABYLON.ArcRotateCamera(
        "camera",
        Math.PI / 2,
        Math.PI / 4,
        10,
        BABYLON.Vector3.Zero(),
        scene
      );
      camera.attachControl(canvas.value, true);
      camera.wheelPrecision = 50; // Чувствительность колесика мыши

      // 3. Добавляем свет
      const light = new BABYLON.HemisphericLight(
        "light",
        new BABYLON.Vector3(0, 1, 0),
        scene
      );
      light.intensity = 0.7;

      // 4. Создаём звёзды (500 белых точек)
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

      // 5. Создаём планету
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

      // 6. Анимация и масштабирование
      scene.registerBeforeRender(() => {
        // Вращение по X и Y
        planet.rotation.x += 0.005; // Наклон
        planet.rotation.y += 0.01; // Вращение

        // Расстояние до планеты
        distance.value = BABYLON.Vector3.Distance(
          camera.position,
          planet.position
        );

        // Уменьшение размера при отдалении (от 100% до 10%)
        const maxDistance = 20; // 10 диаметров планеты
        const scale = Math.max(0.1, 1 - distance.value / maxDistance);
        planetSize.value = scale * 100;
        planet.scaling.setAll(scale);
      });

      // 7. Запускаем анимацию
      engine.runRenderLoop(() => {
        scene.render();
      });

      // 8. Подстраиваемся под размер окна
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
