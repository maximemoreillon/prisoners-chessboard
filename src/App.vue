<script setup lang="ts">
import { computed, ref } from "vue";
import Chessboard from "./components/Chessboard.vue";

const cells = ref(Array.from({ length: 16 }, () => Math.random() >= 0.5));
const groupings = ref([
  {
    color: "lightpink",
    condition: (i: number) => i >= 0.5 * cells.value.length,
  },
  {
    color: "lightgreen",
    condition: (i: number) =>
      Math.floor(i / Math.sqrt(cells.value.length)) % 2 === 0,
  },
  {
    color: "lightblue",
    condition: (i: number) =>
      Math.floor((2 * i) / Math.sqrt(cells.value.length)) % 2 === 0,
  },
  {
    color: "lightsalmon",
    condition: (i: number) =>
      Math.floor((4 * i) / Math.sqrt(cells.value.length)) % 2 === 0,
  },
]);

const parities = computed(() =>
  groupings.value.map(({ condition }) =>
    cells.value.reduce((acc, cell, i) => acc != (cell && condition(i)), true)
  )
);

const coordinateBinary = computed(() => parities.value.map((v) => +v).join(""));

const coordinateDecimal = computed(() => parseInt(coordinateBinary.value, 2));
</script>

<template>
  <div class="wrapper">
    <div>
      <div class="index" style="text-align: left">0</div>
      <Chessboard :cells="cells" :target="coordinateDecimal" />
      <div class="index" style="text-align: right">15</div>
    </div>

    <table>
      <tr>
        <th>Grouping</th>
        <th>Parity</th>
      </tr>
      <tr v-for="({ condition, color }, ci) in groupings" :key="ci">
        <td>
          <Chessboard :cells="cells" :condition="condition" :color="color" />
        </td>
        <td>
          <span
            class="coordinate"
            :style="{ 'background-color': groupings[ci].color }"
          >
            {{ +parities[ci] }}
          </span>
        </td>
      </tr>
    </table>

    <div>
      <span
        v-for="(parity, i) in parities"
        :key="i"
        class="coordinate"
        :style="{
          backgroundColor: groupings[i].color,
          paddingInline: '0.25em',
        }"
      >
        {{ +parity }}
      </span>

      <span> → </span>
      <span class="target coordinate">{{ coordinateDecimal }}</span>
    </div>
  </div>
</template>

<style>
body {
  font-family: Arial, Helvetica, sans-serif;
}
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1em;
}

table {
  text-align: center;
}

.target {
  outline: 2px solid red;
}

.index {
  opacity: 0.5;
  font-size: 80%;
}

.coordinate {
  padding: 0.25em;
}
</style>
