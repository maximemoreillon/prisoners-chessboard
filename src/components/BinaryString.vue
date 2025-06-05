<script setup lang="ts">
import colors from "@/colors";

defineProps<{
  binaryString: string;
  colored?: boolean;
}>();
</script>

<template>
  <span
    v-for="(bit, i) in binaryString"
    :key="i"
    class="coordinate"
    :style="{
      backgroundColor: colored ? colors[i] : undefined,
      paddingInline: '0.25em',
    }"
  >
    {{ +bit }}
  </span>
</template>

<style></style>
