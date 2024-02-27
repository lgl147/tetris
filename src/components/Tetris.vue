<template>
  <div class="container">
    <div
      class="status d-flex justify-space-between align-center"
      :style="'width: ' + width * grid + 'px'"
    >
      <div>分数: {{ score }}</div>
      <div class="next" v-if="currentShape">下一个是: {{ nextShape }}</div>
      <div>
        <v-btn @click="start" variant="outlined">开始</v-btn>
        <v-btn
          v-if="currentShape"
          @click="fallControl"
          size="small"
          :icon="timer ? 'mdi-pause' : 'mdi-play'"
          class="ml-2"
        ></v-btn>
      </div>
    </div>
    <div class="gamearea" :style="'width: ' + width * grid + 'px'">
      <div
        v-for="(row, rowIndex) in dataList"
        :key="rowIndex"
        class="row d-flex"
      >
        <div
          v-for="(col, colIndex) in row"
          :key="colIndex"
          class="col"
          :class="{ solid: col > 0 }"
          :style="`width: ${grid}px;height: ${grid}px`"
        >
          {{ col }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import shapes from "./shapes";
import { onMounted, ref } from "vue";

onMounted(() => {
  initGame();
  bindOperate();
});

let width = 10;
let height = 20;
let grid = ref(40);
let dataList = ref<any>();
let timer = ref<any>(null);
let score = ref<any>(0);

function initGame() {
  nextShape = null;
  clearInterval(timer.value);
  timer.value = null;
  score.value = 0;
  dataList.value = [];
  for (let i = 0; i < height; i++) {
    dataList.value.push(new Array(width).fill(0));
  }
}

function fallControl() {
  if (timer.value) {
    clearInterval(timer.value);
    timer.value = null;
  } else {
    timer.value = setInterval(() => falling(), 500);
  }
}

const shapeNames = Object.keys(shapes);

let currentShape = ref<any>();
let currentShapeName: any = null;
let nextShape: any = null;
function createShape() {
  currentState = 0;
  currentShape.value = null;
  if (nextShape) {
    currentShapeName = nextShape;
  } else {
    currentShapeName = randomShape();
  }
  nextShape = randomShape();

  let startIndex = 4;
  // 有些图形需要往右挪一点儿初始位置才能居中
  if (currentShapeName == "Z" || currentShapeName == "J") startIndex = 5;
  if (currentShapeName == "I") startIndex = 3;
  currentShape.value = shapes[currentShapeName].map((item: any) => {
    return [item[0] + startIndex, item[1]];
  });

  currentShape.value.forEach((item: any) => {
    dataList.value[item[1]][item[0]] = 1;
  });
}

function randomShape() {
  return shapeNames[Math.floor(Math.random() * shapeNames.length)];
}

let scores: any = {
  1: 10,
  2: 30,
  3: 50,
  4: 100,
};

function falling() {
  if (!currentShape.value) return;
  let touchLimit = false;
  currentShape.value.forEach((item: any) => {
    if (item[1] >= height - 1) {
      touchLimit = true;
      return;
    }
    if (dataList.value[item[1] + 1][item[0]] == 2) {
      touchLimit = true;
    }
  });
  if (!touchLimit) {
    let old = JSON.parse(JSON.stringify(currentShape.value));
    currentShape.value.forEach((item: any) => item[1]++);
    landing(currentShape.value, old);
  } else {
    // 成功落地
    currentShape.value.forEach((item: any) => {
      dataList.value[item[1]][item[0]] = 2;
    });
    // 消除
    let line: number = 0;
    dataList.value.forEach((row: any, rowIndex: number) => {
      if (row.every((col: any) => col == 2)) {
        // console.log(`第${rowIndex}行`);
        line++;
        dataList.value.splice(rowIndex, 1);
        dataList.value.unshift(new Array(width).fill(0));
      }
    });
    // 加分
    if (line > 0) score.value += scores[line];
    // 到顶了
    if (dataList.value[0].includes(2)) gameover();
    createShape();
  }
}

function bindOperate() {
  // let touchLimit = false;
  document.onkeydown = (e: KeyboardEvent) => {
    if (!currentShape.value) return;
    let old = JSON.parse(JSON.stringify(currentShape.value));
    let touchLimit = false;

    if (e.key == "ArrowRight") {
      currentShape.value.forEach((item: any) => {
        if (item[0] >= width - 1) {
          touchLimit = true;
        }
        if (dataList.value[item[1]][item[0] + 1] == 2) {
          touchLimit = true;
        }
      });
      if (!touchLimit) currentShape.value.forEach((item: any) => item[0]++);
    }
    if (e.key == "ArrowLeft") {
      currentShape.value.forEach((item: any) => {
        if (item[0] < 1) {
          touchLimit = true;
        }
        if (dataList.value[item[1]][item[0] - 1] == 2) {
          touchLimit = true;
        }
      });
      if (!touchLimit) currentShape.value.forEach((item: any) => item[0]--);
    }
    if (e.key == "ArrowDown") {
      falling();
      return;
    }
    if (e.key == "ArrowUp") {
      rotate();
    }

    if (e.key == "c") {
      nextShape = "I";
    }

    if (!touchLimit) landing(currentShape.value, old);
  };
}

let currentState = 0;

let states: any = {
  I: [
    [
      [1, -1],
      [0, 0],
      [-1, 1],
      [-2, 2],
    ],
    [
      [-1, 1],
      [0, 0],
      [1, -1],
      [2, -2],
    ],
  ],
  S: [
    [
      [1, 0],
      [0, -1],
      [-1, 0],
      [-2, -1],
    ],
    [
      [-1, 0],
      [0, 1],
      [1, 0],
      [2, 1],
    ],
  ],
  Z: [
    [
      [-2, 0],
      [-1, -1],
      [0, 0],
      [1, -1],
    ],
    [
      [2, 0],
      [1, 1],
      [0, 0],
      [-1, 1],
    ],
  ],
  T: [
    [
      [1, 1],
      [1, -1],
      [0, 0],
      [-1, 1],
    ],
    [
      [-1, 1],
      [1, 1],
      [0, 0],
      [-1, -1],
    ],
    [
      [-1, -1],
      [-1, 1],
      [0, 0],
      [1, -1],
    ],
    [
      [1, -1],
      [-1, -1],
      [0, 0],
      [1, 1],
    ],
  ],
  L: [
    [
      [1, 1],
      [0, 0],
      [-1, -1],
      [-2, 0],
    ],
    [
      [-1, 1],
      [0, 0],
      [1, -1],
      [0, -2],
    ],
    [
      [-1, -1],
      [0, 0],
      [1, 1],
      [2, 0],
    ],
    [
      [1, -1],
      [0, 0],
      [-1, 1],
      [0, 2],
    ],
  ],
  J: [
    [
      [1, 1],
      [0, 0],
      [-1, -1],
      [0, -2],
    ],
    [
      [-1, 1],
      [0, 0],
      [1, -1],
      [2, 0],
    ],
    [
      [-1, -1],
      [0, 0],
      [1, 1],
      [0, 2],
    ],
    [
      [1, -1],
      [0, 0],
      [-1, 1],
      [-2, 0],
    ],
  ],
};

function rotate() {
  if (!currentShapeName || !currentShape.value) return;
  if (currentShapeName == "O") return;
  let hasRepeat = false;
  let touchLeft = false;
  let touchTop = false;
  let touchRight = false;
  states[currentShapeName][currentState].forEach((item: any, index: any) => {
    let newPosition: any = [
      currentShape.value[index][0] + item[0],
      currentShape.value[index][1] + item[1],
    ];
    if (newPosition[0] < 0) {
      touchLeft = true;
      newPosition[0] = 0;
    }
    if (newPosition[1] < 0) {
      touchTop = true;
      newPosition[1] = 0;
    }
    if (newPosition[0] > width - 1) {
      touchRight = true;
      newPosition[0] = width - 1;
    }
    if (dataList.value[newPosition[1]][newPosition[0]] == 2) hasRepeat = true;
  });
  if (hasRepeat) return;
  states[currentShapeName][currentState].forEach((item: any, index: any) => {
    currentShape.value[index][0] += item[0];
    currentShape.value[index][1] += item[1];

    if (touchLeft) {
      currentShape.value[index][0] += 1;
    }
    if (touchTop) {
      currentShape.value[index][1] += 1;
    }
    if (touchRight) {
      if (currentShapeName == "I") currentShape.value[index][0] -= 2;
      else currentShape.value[index][0] -= 1;
    }
  });
  currentState++;
  if (currentState > states[currentShapeName].length - 1) currentState = 0;
}

function landing(newPosition: any, oldPosition: any) {
  if (!newPosition || !oldPosition) return;
  oldPosition.forEach((item: any) => {
    dataList.value[item[1]][item[0]] = 0;
  });
  newPosition.forEach((item: any) => {
    dataList.value[item[1]][item[0]] = 1;
  });
}

function start() {
  initGame();
  fallControl();
  createShape();
}

function gameover() {
  clearInterval(timer.value);
  timer.value = null;

  alert("游戏结束");
}
</script>

<style lang="scss" scoped>
.container {
  padding: 20px;
  .gamearea {
    // width: 400px;
    border: 0.5px solid;
    .col {
      // box-sizing: border-box;
      border: 0.5px solid;
      display: inline-flex;
      justify-content: center;
      align-items: center;
      transition: all 0.1s;
    }
    .solid {
      background-color: red;
    }
  }
  .btns {
    position: fixed;
    right: 20px;
    top: 20px;
    .v-btn {
      margin-bottom: 10px;
    }
  }
}
.status {
  height: 64px;
}
</style>
