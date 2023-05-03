<template>
  <div class="board">
    <div class="row" v-for="r in height" :key="`r_${r}`">
      <div class="box" v-for="c in width" :key="`c_${c}`"
        :class="{ isClicked: state.cards[(r - 1) * width + c - 1].flag }" @click="handleClickCard(r, c)">{{ state.cards[(r
          - 1) * width + c - 1
        ].value }}</div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive } from 'vue';

const pairs = 6
const state = reactive({
  cards: []
})
// 这个输入对数，会算出对应个数最近似正方形的宽高边个数，宽 > 高
// eg: 12 => [4, 3]
const getXYByPairs = (pairs) => {
  // const count = pairs * 2
  const total = pairs * 2
  let width = pairs
  let res = width
  while (width > 0) {
    width--
    if (total % width !== 0) continue
    if ((total / width) > width) break
    res = width
  }
  return [res, total / res]
}
// 在数组里随机安排n对
const genRandomCards = (pairs) => {
  // 初始化一个有两个pairs的数组, flag用于记录是否已经被翻牌，已翻的牌再次点击不生效
  const initNumber = Array.from({ length: pairs }).map((_, index) => { return { value: index + 1, flag: false } })
  const cards = [...initNumber, ...initNumber];

  // 随机取一个数，和最后一个交换，然后改变length 重复操作
  // const index = Math.floor(Math.random() * total)
  let len = cards.length - 1;
  while (len > 0) {
    const target = Math.floor(Math.random() * len);
    [cards[target], cards[len]] = [cards[len], cards[target]];
    len--;
  }
  return cards;
}
const getCardNumber = (row, column) => {
  return state.cards[(row - 1) * width + column - 1];
}

let currentRound = []; // 每两次点击，是一个round, 记录第一次点击的坐标
// 每两次
const handleClickCard = (row, column) => {
  const currentNumber = getCardNumber(row, column);
  // 如果已经翻牌了，本次点击无效
  if (currentNumber.flag) return;
  // 没有点击过，记录pre点击
  if (currentRound.length === 0) {
    currentRound = [row, column];
  } else {
    const pre = getCardNumber(currentRound[0], currentRound[1]);

    if (pre.value === currentNumber.value) {
      console.log('=====bingo=====');
      pre.flag = true;
      currentNumber.flag = true;
    }
    // new round
    currentRound = [];
  }
}

// 返回
const [width, height] = getXYByPairs(pairs)
state.cards = genRandomCards(pairs);

</script>

<style scoped>
.board {
  display: flex;
  flex-direction: column;
}

.board .row {
  display: flex;
}

.board .row .box {
  width: 200px;
  height: 200px;
  border: 1px solid black;
}

.board .row .box.isClicked {
  background-color: red;
}

.board .row .box+.board .row .box {
  margin-left: 10px;
}
</style>