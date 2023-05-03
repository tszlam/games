<template>
  <input type="number" v-model="pairs"/>
  <button @click="restart">New Game</button>
  <div>STAET: {{ isFinished ? 'FINISHED' : 'UN FINISH' }}</div>
  <div class="board">
    <div class="row" v-for="r in board.row" :key="`r_${r}`">
      <div class="box" v-for="c in board.column" :key="`${r}_${c}`"
        :class="{
          isClicked: getCardNumber(r, c).state === CARD_STATE.PAIRING,
          isPaired: getCardNumber(r, c).state === CARD_STATE.PAIRED,
        }"
        @click="handleClickCard(r, c)"
      >
        {{ getCardNumber(r, c).value }}
      </div>
    </div>
  </div>
</template>

<script setup>
// todo: 开始时倒数，倒数是可以看数字，倒数结束后开始游戏，游戏是不可以看数字，只有激发态才可以看到数字
import { ref, reactive, computed } from 'vue';
const CARD_STATE = {
  PENDING: 0,
  PAIRING: 1,
  PAIRED: 2,
}
const numPerPair = 2
const pairs = ref(6)
const state = reactive({
  cards: []
})
let pairingStack = [];
// 这个输入对数，会算出对应个数最近似正方形的宽高边个数，宽 > 高
// eg: 12 => [4, 3]
/**
 * 这个输入对数，会算出对应个数最近似正方形的行数和列数，列 > 行
 * eg: 12 => { row: 3, column: 4 }
 * @param {number} pairs 
 */
const getXYByPairs = (pairs) => {
  const total = pairs * 2
  let width = pairs
  let res = width
  while (width > 0) {
    width--
    if (total % width !== 0) continue
    if ((total / width) > width) break
    res = width
  }
  return {
    row: total / res,
    column: res,
  }
}
// 在数组里随机安排n对
const genRandomCards = (pairs) => {
  // 初始化一个有两个pairs的数组, flag用于记录是否已经被翻牌，已翻的牌再次点击不生效
  const length = pairs * numPerPair
  // value生成
  let value = 0
  let count = 0
  const getValue = () => {
      if (count === 0 ) {
        count = numPerPair
        value++
      }
      count--
      return value
  }
  const cards = Array
    .from({ length })
    .map(() => ({ value: getValue(), flag: false, state: CARD_STATE.PENDING }));

  // 随机取一个数，和最后一个交换，然后改变length 重复操作
  let len = length - 1;
  while (len > 0) {
    const target = Math.floor(Math.random() * (len + 1)); // random是[0,1) 所以要把len+1
    if (len !== target) {
      [cards[target], cards[len]] = [cards[len], cards[target]];
    }
    len--;
  }
  return cards;
}
const getCardNumber = (row, column) => {
  return state.cards[(row - 1) * board.column + column - 1];
}

const handleClickCard = (row, column) => {
  const card = getCardNumber(row, column);
  const { value, state } = card
  if (state !== CARD_STATE.PENDING) return;
  if (pairingStack.length === 0) {
    card.state = CARD_STATE.PAIRING
    pairingStack.push(card)
    return;
  }
  if (pairingStack[0].value !== value) {
    pairingStack.forEach(_card => {
      _card.state = CARD_STATE.PENDING
    })
    pairingStack = []
    return
  }
  if (pairingStack.length + 1 < numPerPair) {
    card.state = CARD_STATE.PAIRING
    pairingStack.push(card)
  }
  card.state = CARD_STATE.PAIRED
  pairingStack.forEach(_card => {
    _card.state = CARD_STATE.PAIRED
  })
  pairingStack = []
}

const restart = () => {
  board = reactive(getXYByPairs(pairs.value))
  state.cards = genRandomCards(pairs.value)
  pairingStack = []
}

// 返回
let board = reactive(getXYByPairs(pairs.value))
state.cards = genRandomCards(pairs.value);
const isFinished = computed(() => state.cards.every(({ state: _s }) => _s === CARD_STATE.PAIRED ))

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
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bolder;
  font-size: 40px;
}

.board .row .box.isClicked {
  background-color: red;
  color: white;
}.board .row .box.isPaired {
  background-color: green;
  color: white;
}

.board .row .box+.board .row .box {
  margin-left: 10px;
}
</style>