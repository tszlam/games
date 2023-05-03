<template>
  <div>
  Num Per Pair:<input type="number" v-model="numPerPair"/>
  Pairs: <input type="number" v-model="pairs"/>
  Seconds: <input type="number" v-model="secToMemorize"/>
  Froze: <input type="number" v-model="secForze">
  <button :disabled="isPlaying" @click="start">New Game</button>
  <button :disabled="!isPlaying" @click="end">End Game</button>
  </div>
  <div v-if="isPlaying">STAET: {{ isFinished ? 'FINISHED' : 'UN FINISH' }}</div>
  <div v-else>{{ secToPlay ? `${secToPlay} sec to start` : '' }}</div>
  <div class="board">
    <div class="row" v-for="r in board.row" :key="`r_${r}`">
      <div class="box" v-for="c in board.column" :key="`${r}_${c}`"
        :class="{
          isClicked: getCardNumber(r, c).state === CARD_STATE.PAIRING,
          isPaired: getCardNumber(r, c).state === CARD_STATE.PAIRED,
        }"
        @click="handleClickCard(r, c)"
      >
        <span v-show="getCardNumber(r, c).state !== CARD_STATE.PENDING || !isPlaying">{{ getCardNumber(r, c).value }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
// GAME STATE: SETTING -> MEMORIZING -> MATCHING -> [FROZE -> MATCHING] -> FINISHED/END
// todo: 游戏的状态，操作逻辑怎么管理
import { ref, reactive, computed } from 'vue';
const CARD_STATE = {
  PENDING: 0,
  PAIRING: 1,
  PAIRED: 2,
}
// const GAME_STATE = {
//   SETTING: 0,
//   MEMORIZING: 1,
//   MATCHING: 2,
//   FROZE: 3,
//   FINISHED: 4,
//   END: 5
// }
const numPerPair = ref(2)
const pairs = ref(6)
const secToMemorize = ref(5)
const secToPlay = ref(0)
const secForze = ref(0.5)
const isPlaying = ref(false)
const isChecking = ref(false)
const state = reactive({
  cards: genRandomCards(0)
})
const isFinished = computed(() => state.cards.every(({ state: _s }) => _s === CARD_STATE.PAIRED ))
let pairingStack = [];
let board = reactive(getXYByPairs(0))

/**
 * 这个输入对数，会算出对应个数最近似正方形的行数和列数，列 > 行
 * eg: 12 => { row: 3, column: 4 }
 * @param {number} pairs 
 */
function getXYByPairs (pairs) {
  if (!pairs) return { row: 0, column: 0 };
  const total = pairs * numPerPair.value
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
function genRandomCards(pairs) {
  // 初始化一个有两个pairs的数组, flag用于记录是否已经被翻牌，已翻的牌再次点击不生效
  const length = pairs * numPerPair.value
  // value生成
  let value = 0
  let count = 0
  const getValue = () => {
      if (count === 0 ) {
        count = numPerPair.value
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
function getCardNumber(row, column) {
  return state.cards[(row - 1) * board.column + column - 1];
}

function handleClickCard(row, column) {
  if (!isPlaying.value) return
  if (isChecking.value) return
  const card = getCardNumber(row, column);
  const { value, state } = card
  if (state !== CARD_STATE.PENDING) return;
  if (pairingStack.length === 0) {
    card.state = CARD_STATE.PAIRING
    pairingStack.push(card)
    return;
  }
  if (pairingStack[0].value !== value) {
    isChecking.value = true
    card.state = CARD_STATE.PAIRING
    pairingStack.push(card)
    setTimeout(() => {
      pairingStack.forEach(_card => {
        _card.state = CARD_STATE.PENDING
      })
      pairingStack = []
      isChecking.value = false
    }, secForze.value * 1000)
    return
  }
  if (pairingStack.length + 1 < numPerPair.value) {
    card.state = CARD_STATE.PAIRING
    pairingStack.push(card)
  }
  card.state = CARD_STATE.PAIRED
  pairingStack.forEach(_card => {
    _card.state = CARD_STATE.PAIRED
  })
  pairingStack = []
}

function countDown(sec) {
  secToPlay.value = sec
  if (sec === 0) {
    isPlaying.value = true
    return
  }
  setTimeout(() => {
    countDown(secToPlay.value - 1)
  }, 1000)
}

function start() {
  board = reactive(getXYByPairs(pairs.value))
  state.cards = genRandomCards(pairs.value)
  pairingStack = []
  isPlaying.value = false
  countDown(secToMemorize.value >= 0 ? secToMemorize.value : 0)
}
function end() {
  isPlaying.value = false
}
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