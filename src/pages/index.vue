<script setup lang='ts'>

const WIDTH = 15
const HEIGHT = 15
const gaming = ref(false)
const data: Array<Array<BlockState>> = reactive([])
interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine: boolean
  flagged: boolean
  adjacentMines: number
}
const directions = [
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, -1],
  [0, 1],
  [1, -1],
  [1, 0],
  [1, 1],
]

const numberColors = [
  'text-transparent',
  'text-gray-500',
  'text-green-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-yellow-500',
  'text-blue-500',
]
function onRightClick(block: BlockState) {
  if (block.revealed) return
  block.flagged = !block.flagged
}

function expandZero(block: BlockState) {
  directions.forEach(([dx, dy]) => {
    const x = block.x + dx
    const y = block.y + dy
    if (!isValid(x, y)) return
    const block2 = data[x][y]
    if (!block2.mine && !block2.revealed) {
      block2.revealed = true
      if (block2.adjacentMines === 0)
        expandZero(block2)
    }
  })
}

function onClick(block: BlockState) {
  if (!gaming.value) {
    gaming.value = true
    generateMines(block)
    updateNumbers()
  }
  if (!block.revealed) {
    block.revealed = !block.revealed
    if (block.mine) {
      gaming.value = false
      return alert('你输了！')
    }
    if (!block.mine && block.adjacentMines === 0)expandZero(block)
  }
}

function initData() {
  for (let i = 0; i < HEIGHT; i++) {
    const row: Array<BlockState> = []
    for (let j = 0; j < WIDTH; j++) {
      const blockState: BlockState = {
        x: i,
        y: j,
        revealed: false,
        mine: false,
        flagged: false,
        adjacentMines: 0,
      }
      row.push(blockState)
    }
    data.push(row)
  }
}

function generateMines(initial: BlockState) {
  for (const row of data) {
    for (const block of row)
      block.mine = Math.random() < 0.12
  }
  initial.mine = false
}

function getBlockClass(block: BlockState) {
  if (!block.revealed) return 'bg-gray-500/10'
  if (block.mine) return 'text-red'
  return numberColors[block.adjacentMines]
}
function isValid(x: number, y: number) {
  if (x < 0 || x >= WIDTH) return false
  if (y < 0 || y >= HEIGHT) return false
  return true
}

function updateNumbers() {
  data.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) return
      directions.forEach(([dx, dy]) => {
        const xx = x + dx
        const yy = y + dy
        if (isValid(xx, yy) && data[yy][xx].mine)
          block.adjacentMines++
      })
    })
  })
}
initData()
</script>

<template>
  <div p-5>
    <div>
      Minesweeper {{ gaming? '游戏中': '未开始' }}
    </div>
    <div
      v-for="row, x in data"
      :key="x"
      flex="~"
      justify-center
      items-center
    >
      <button
        v-for="block, y in row"
        :key="y"
        w-10 h-10 m-1
        flex="~"
        justify-center
        items-center
        border="1 gray-500/20"
        :class="getBlockClass(block)"
        hover="bg-gray-500/20"
        @click="onClick(block)"
        @contextmenu.prevent="onRightClick(block)"
      >
        <div v-if="block?.revealed">
          <div v-if="block?.mine" i-mdi-mine />
          <div v-else>
            {{ block.adjacentMines }}
          </div>
        </div>
        <div v-else-if="block.flagged" i-mdi:flag-variant text-red>
          <div />
        </div>
      </button>
    </div>
  </div>
</template>
