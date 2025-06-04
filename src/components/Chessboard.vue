<script setup lang="ts">
const props = defineProps<{
  size: number;
  cells: Boolean[];
  condition?: Function;
  color?: string;
  target?: number;
}>();

const styleOfCell = (cellIndex: number) => ({
  backgroundColor:
    props.condition && props.condition(cellIndex)
      ? props.color || "lightpink"
      : "undefined",
});

const classOfCell = (cellIndex: number) => ({
  target: cellIndex === props.target,
});
</script>

<template>
  <div
    class="chessboard"
    :style="{ gridTemplateColumns: `repeat(${Math.sqrt(props.size)}, auto)` }"
  >
    <div
      v-for="(_, i) in props.cells"
      :key="i"
      :style="styleOfCell(i)"
      :class="classOfCell(i)"
    >
      <input type="checkbox" v-model="props.cells[i]" />
    </div>
  </div>
</template>

<style scoped>
.chessboard {
  margin: 0.25em;
  border: 1px solid black;
  display: grid;
}

.chessboard > div {
  display: flex;
  align-items: center;
  justify-self: center;
}

input[type="checkbox"] {
  width: 1.5em;
  height: 1.5em;
}

.green {
  background-color: lightgreen;
}

.red {
  background-color: lightpink;
}
</style>
