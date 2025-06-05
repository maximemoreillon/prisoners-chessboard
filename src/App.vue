<script setup lang="ts">
import { computed, ref } from "vue";
import Chessboard from "./components/Chessboard.vue";
import colors from "@/colors";
import BinaryString from "./components/BinaryString.vue";

function generateCells(length: number) {
  return Array.from({ length }, () => Math.random() >= 0.5);
}

function changeBoardSize() {
  if (boardSize.value === 16) cells.value = generateCells(64);
  else cells.value = generateCells(16);
}

const cells = ref(generateCells(16));

const boardSize = computed(() => cells.value.length);
const groupingsCount = computed(() => boardSize.value.toString(2).length - 1);
const target = computed(() => Math.round(Math.random() * boardSize.value));

const groupingConditions = computed(() =>
  Array.from(
    { length: groupingsCount.value },
    (_, ci) => (i: number) => i.toString(2).at(-ci - 1) === "1"
  )
);

// Parities for each grouping
const parities = computed(() =>
  groupingConditions.value.map((condition) =>
    cells.value.reduce((acc, cell, i) => acc != (cell && condition(i)), true)
  )
);

const paritiesBinary = computed(() =>
  parities.value
    .map((p) => +p)
    .join("")
    .padStart(groupingsCount.value, "0")
);

function bin2Dec(bin: string) {
  return parseInt(bin, 2);
}

function dec2Bin(dec: number) {
  return dec.toString(2).padStart(groupingsCount.value, "0");
}

const paritiesDecimal = computed(() => bin2Dec(paritiesBinary.value));
const difference = computed(() => target.value ^ paritiesDecimal.value);

const coinToFlip = computed(() => {
  const matchedConditions = groupingConditions.value.filter(
    (_, i) => dec2Bin(difference.value)[i] === "1"
  );

  return cells.value.findIndex((_, ci) =>
    matchedConditions.every((mc) => mc(ci))
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
        :current="paritiesDecimal"
        :size="boardSize"
      />
      <div class="index">
        <div>{{ boardSize - 1 - Math.sqrt(boardSize) }}</div>
        <div>{{ boardSize - 1 }}</div>
      </div>
    </div>

    <table class="groupings">
      <thead>
        <tr>
          <th>Grouping</th>
          <th>Parity</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(_, i) in groupingConditions" :key="i">
          <td>
            <Chessboard
              :cells="cells"
              :condition="groupingConditions[i]"
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

    <table class="calculations">
      <thead>
        <tr>
          <th></th>
          <th>Binary</th>
          <th>Decimal</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Parities</td>
          <td>
            <BinaryString :binaryString="paritiesBinary" colored />
          </td>
          <td>
            <span class="current coordinate">
              {{ paritiesDecimal }}
            </span>
          </td>
        </tr>
        <tr>
          <td>Target</td>
          <td>
            <BinaryString :binaryString="dec2Bin(target)" />
          </td>
          <td>
            <span class="target coordinate">
              {{ target }}
            </span>
          </td>
        </tr>

        <tr>
          <td>XOR</td>
          <td>
            <BinaryString :binaryString="dec2Bin(difference)" colored />
          </td>
          <td></td>
        </tr>

        <tr>
          <td>To flip</td>
          <td>
            <BinaryString :binaryString="dec2Bin(coinToFlip)" />
          </td>
          <td>
            <span class="toFlip coordinate">
              {{ coinToFlip }}
            </span>
          </td>
        </tr>
      </tbody>
    </table>
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
  border-collapse: collapse;
}

.calculations td {
  padding: 0.5em;
}

.coordinate {
  padding: 0.25em;
}

.current {
  outline: 1px dashed red;
}

.toFlip {
  outline: 1px solid blue;
}

.target {
  outline: 2px solid green;
}

.index {
  opacity: 0.5;
  font-size: 80%;
  display: flex;
  justify-content: space-between;
}
</style>
