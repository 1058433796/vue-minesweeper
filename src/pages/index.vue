<script setup lang="ts">

function onClick(x: number, y: number) {
  console.log(x, y)
}
</script>

<template>
  <div>
    Minesweeper
    <div v-for="x in 10" :key="x">
      <button
        v-for="y in 10"
        :key="y"
        w-10 h-10 border
        hover:bg-gray
        @click="onClick(x, y)"
      >
        {{ (x - 1)*10 + y }}
      </button>
    </div>
  </div>
</template>
