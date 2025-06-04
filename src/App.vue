<script setup lang="ts">
import { computed, ref } from "vue";
import Chessboard from "./components/Chessboard.vue";

const boardSize = ref<16 | 64>(16);

const cells = ref(
  Array.from({ length: boardSize.value }, () => Math.random() >= 0.5)
);

const colors = [
  "lightpink",
  "lightblue",
  "lightgreen",
  "lightsalmon",
  "aquamarine",
  "lightgrey",
];

const conditions = ref(
  Array.from(
    { length: boardSize.value.toString(2).length - 1 },
    (_, ci) => (i: number) => i.toString(2).at(-ci - 1) === "1"
  )
);

const parities = computed(() =>
  conditions.value.map((condition) =>
    cells.value.reduce((acc, cell, i) => acc != (cell && condition(i)), true)
  )
);

const coordinateBinary = computed(() => parities.value.map((v) => +v).join(""));
const coordinateDecimal = computed(() => parseInt(coordinateBinary.value, 2));
</script>

<template>
  <div class="wrapper">
    <div>
      <div class="index">
        <div>0</div>
        <div>{{ Math.sqrt(boardSize) }}</div>
      </div>
      <Chessboard
        :cells="cells"
        :target="coordinateDecimal"
        :size="boardSize"
      />
      <div class="index">
        <div>{{ boardSize - 1 - Math.sqrt(boardSize) }}</div>
        <div>{{ boardSize - 1 }}</div>
      </div>
    </div>

    <table>
      <thead>
        <tr>
          <th>Grouping</th>
          <th>Parity</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(_, i) in conditions" :key="i">
          <td>
            <Chessboard
              :cells="cells"
              :condition="conditions[i]"
              :color="colors[i]"
              :size="boardSize"
            />
          </td>
          <td>
            <span class="coordinate" :style="{ 'background-color': colors[i] }">
              {{ +parities[i] }}
            </span>
          </td>
        </tr>
      </tbody>
    </table>

    <div>
      <span
        v-for="(parity, i) in parities"
        :key="i"
        class="coordinate"
        :style="{
          backgroundColor: colors[i],
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
  display: flex;
  justify-content: space-between;
}

.coordinate {
  padding: 0.25em;
}
</style>
