<script setup lang="ts">
import { computed, ref } from "vue";
import Chessboard from "./components/Chessboard.vue";

function generateCells(length: number) {
  return Array.from({ length }, () => Math.random() >= 0.5);
}
const cells = ref(generateCells(16));

const boardSize = computed(() => cells.value.length);

function changeBoardSize() {
  if (boardSize.value === 16) cells.value = generateCells(64);
  else cells.value = generateCells(16);
}

const groupingsCount = computed(() => boardSize.value.toString(2).length - 1);

const target = ref(Math.round(Math.random() * boardSize.value));

const colors = [
  "lightpink",
  "lightblue",
  "lightgreen",
  "lightsalmon",
  "aquamarine",
  "lightgrey",
];

const conditions = computed(() =>
  Array.from(
    { length: groupingsCount.value },
    (_, ci) => (i: number) => i.toString(2).at(-ci - 1) === "1"
  )
);

const parities = computed(() =>
  conditions.value.map((condition) =>
    cells.value.reduce((acc, cell, i) => acc != (cell && condition(i)), true)
  )
);

const paritiesBinary = computed(() =>
  parities.value
    .map((p) => +p)
    .join("")
    .padStart(groupingsCount.value, "0")
);

const coordinateBinary = computed(() =>
  parities.value
    .map((v) => +v)
    .join("")
    .padStart(groupingsCount.value, "0")
);
const coordinateDecimal = computed(() => parseInt(coordinateBinary.value, 2));

const targetBinary = computed(() =>
  target.value.toString(2).padStart(groupingsCount.value, "0")
);
const difference = computed(() => target.value ^ coordinateDecimal.value);

const differenceBinary = computed(() =>
  difference.value.toString(2).padStart(groupingsCount.value, "0")
);

// Does not work
const coinToFlip = computed(() => {
  const matchedConditions = conditions.value.filter(
    (_, i) => differenceBinary.value[i] === "1"
  );

  return cells.value.findIndex(
    (_, ci) => !matchedConditions.some((mc) => !mc(ci))
  );
});
</script>

<template>
  <div class="wrapper">
    <div>
      <button @click="changeBoardSize">Board size: {{ boardSize }}</button>
    </div>
    <div>
      <div class="index">
        <div>0</div>
        <div>{{ Math.sqrt(boardSize) - 1 }}</div>
      </div>
      <Chessboard
        :cells="cells"
        :target="target"
        :toFlip="coinToFlip"
        :current="coordinateDecimal"
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
      <h2>Current</h2>
      <span
        v-for="(parity, i) in paritiesBinary"
        :key="i"
        class="coordinate"
        :style="{
          backgroundColor: colors[i],
          paddingInline: '0.25em',
        }"
      >
        {{ parity }}
      </span>

      <span> → </span>
      <span class="current coordinate">{{ coordinateDecimal }}</span>
    </div>
    <div>
      <h2>Target</h2>
      <span
        v-for="(bit, i) in targetBinary"
        :key="i"
        class="coordinate"
        :style="{
          backgroundColor: colors[i],
          paddingInline: '0.25em',
        }"
      >
        {{ bit }}
      </span>

      <span> ← </span>
      <span class="target coordinate">{{ target }}</span>
    </div>

    <div>
      <h2>Difference</h2>
      <span
        v-for="(bit, i) in differenceBinary"
        :key="i"
        class="coordinate"
        :style="{
          backgroundColor: colors[i],
          paddingInline: '0.25em',
        }"
      >
        {{ +bit }}
      </span>
    </div>
    <div>
      <h2>Coin to flip</h2>
      <div style="text-align: center">
        <span class="toFlip coordinate">{{ coinToFlip }}</span>
      </div>
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
  padding: 1em;
}

table {
  text-align: center;
}

.coordinate {
  padding: 0.25em;
}

.current {
  outline: 2px dashed red;
}

.toFlip {
  outline: 2px dotted blue;
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
</style>
