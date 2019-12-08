<template>
  <div id="app">
    <div class="container">
      <div class="head">
        <button
          class="play-bot"
          v-on:click="hasBot = !hasBot; init()">
          {{ hasBot ? "Jouer contre un(e) ami(e)" : "Jouer contre l'ordinateur" }}
        </button>
        <button>
          <svg
            class="symbol symbol-x"
            aria-label="X"
            role="img"
            viewBox="0 0 128 128">
            <path d="M16,16L112,112"></path>
            <path d="M112,16L16,112"></path>
          </svg>
          <span class="nb-win">{{x}}</span>
        </button>
        <button
          v-on:click="hasBot ? init() & playBot() : null">
          <svg
            class="symbol symbol-o"
            aria-label="O"
            role="img"
            viewBox="0 0 128 128">
            <path d="M64,16A48,48 0 1,0 64,112A48,48 0 1,0 64,16"></path>
          </svg>
          <span class="nb-win">{{o}}</span>
        </button>
      </div>
      <div
        v-if="showResult"
        class="result">
        <div class="result-value">
          <svg
            v-if="!result || result === 1"
            class="symbol symbol-x"
            aria-label="X"
            role="img"
            viewBox="0 0 128 128">
            <path d="M16,16L112,112"></path>
            <path d="M112,16L16,112"></path>
          </svg>
          <svg
            v-if="!result || result === 2"
            class="symbol symbol-o"
            aria-label="O"
            role="img"
            viewBox="0 0 128 128">
            <path d="M64,16A48,48 0 1,0 64,112A48,48 0 1,0 64,16"></path>
          </svg>
        </div>
        <div class="result-name">
          <span v-if="!result">Egalité !</span>
          <span v-else>Gagné !</span>
        </div>
      </div>
      <div
        v-else
        class="game">
        <div
          v-for="(row, ri) in values"
          :key="ri"
          class="row">
          <button
            v-for="(col, ci) in row"
            :key="ci"
            v-on:click="playerPlay(ri, ci)"
            :disabled="isDisabled(ri, ci)"
            class="col"
            :class="`col-${col}`">
            <transition name="symbol-x">
              <svg
                v-if="col === 'x'"
                class="symbol symbol-x"
                aria-label="X"
                role="img"
                viewBox="0 0 128 128">
                <path d="M16,16L112,112"></path>
                <path d="M112,16L16,112"></path>
              </svg>
            </transition>
            <transition name="symbol-o">
              <svg
                v-if="col === 'o'"
                class="symbol symbol-o"
                aria-label="O"
                role="img"
                viewBox="0 0 128 128">
                <path d="M64,16A48,48 0 1,0 64,112A48,48 0 1,0 64,16"></path>
              </svg>
            </transition>
          </button>
        </div>
      </div>
      <button class="replay" v-on:click="init()">
        Rejouer
      </button>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'


export default {
  name: 'app',
  data () {
    return {
      o: 0,
      x: 0,
      turn: null,
      hasBot: true,
      isFinish: false,
      showResult: false,
      isWait: false,
      nbCase: 3,
      values: null,
    }
  },
  methods: {
    isDisabled (row, col) {
      return this.isWait || this.values[row][col] !== null || this.isFinish
    },
    changeTurn () {
      this.turn = this.turn === 1 ? 2 : 1
    },
    play (row, col) {
      Vue.set(this.values[row], col, this.turn === 1 ? 'x' : 'o')
      this.checkFinish()
    },
    playerPlay (row, col) {
      if (this.isDisabled(row, col))
        return

      this.play(row, col)

      if (this.isFinish) return

      this.changeTurn()
      if (this.hasBot) {
        this.isWait = setTimeout(() => {
          this.playBot()
          this.isWait = null
        }, 500)
      }
    },
    playBot () {
      const values = []
      this.values.forEach((row, ri) => {
        row.forEach((col, ci) => {
          if (col === null)
            values.push(`${ri}-${ci}`)
        })
      })

      const value = values[Math.floor(Math.random() * values.length)].split('-')

      this.play(value[0], value[1])

      if (this.isFinish) return

      this.changeTurn()
    },
    finish (result) {
      if (typeof result === 'undefined') {
        result = this.turn
        if (result === 1)
          this.x++
        else
          this.o++
      }
      this.result = result
      this.isFinish = true
      setTimeout(() => this.showResult = true, 2000)
    },
    checkFinish () {

      for (let i = 0; i < this.nbCase; i++) {
        values = []
        for (let j = 0; j < this.nbCase; j++) {
          values.push(this.values[i][j])
        }
        if (this.isEgual(values))
          return this.finish()
      }

      for (let i = 0; i < this.nbCase; i++) {
        values = []
        for (let j = 0; j < this.nbCase; j++) {
          values.push(this.values[j][i])
        }
        if (this.isEgual(values))
          return this.finish()
      }

      values = []
      for (let i = 0; i < this.nbCase; i++)
        values.push(this.values[i][i])

      if (this.isEgual(values))
        return this.finish()

      values = []
      for (let i = 0; i < this.nbCase; i++)
        values.push(this.values[i][(this.nbCase - 1) - i])

      if (this.isEgual(values))
        return this.finish()

      let values = []
      this.values.forEach(row => {
        row.forEach(col => {
          if (col === null)
            values.push(true)
        })
      })

      if (!values.length)
        return this.finish(null)
    },
    isEgual(data) {
      if (data.every(value => value === 'x'))
        return true
      return data.every(value => value === 'o')
    },
    init () {
      this.turn = 1
      this.isFinish = false
      this.showResult = false
      this.values = []
      for (let i = 0; i < this.nbCase; i++) {
        const row = []
        this.values.push(row)
        for (let j = 0; j < this.nbCase; j++)
          row.push(null)
      }
    },
  },
  created () {
    this.init()
  }
}
</script>

<style>
body {
  margin: 0;
  background: #dadada;
  font-family: sans-serif;
}

#app, .container, .head, .game, .result, .row, .col {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: inherit;
}


#app {
  width: 100vw;
  height: 100vh;
  background-color: #26A69A;
}

button {
  outline: none;
}

.container {
  flex-direction: column;
}

.symbol {
  stroke-width: 12;
  fill: transparent;
  stroke: #000;
  width: 200px;
}

.symbol-o {
  stroke: #fff;
}

.head {
  margin-bottom: 2rem;
  width: 100%;
  justify-content: flex-start;
}

.head button {
  cursor: pointer;
  border: none;
  background-color: #B2DFDB;
  height: 40px;
  margin-left: 8px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.head .nb-win {
  font-size: 1.8rem;
  margin-left: 2rem;
  color: #9e9e9e;
}

.head .play-bot {
  flex: 1;
  text-transform: uppercase;
  font-weight: 600;
  color: #9e9fa1;
  margin-left: 0;
}

.head .symbol {
  width: 24px;
}

.result div {
  font-size: 4rem;
  margin-bottom: 4rem;
  text-transform: uppercase;
  font-weight: 600;
}

.result, .game {
  width: 600px;
  height: 600px;
  flex-direction: column;
}

.replay {
  cursor: pointer;
  width: 100%;
  border: none;
  margin-top: 2rem;
  padding: 1rem;
  text-transform: uppercase;
  font-weight: 800;
  color: #9E9E9E;
  border-radius: 8px;
  background-color: #FAFAFA;
  font-size: 1rem;
}

.row, .col {
  flex: 0 33%;
}

.row {
  width: 100%;
}

.col,
.col[disabled] {
  cursor: pointer;
  border: 4px solid #00897B;
  height: 100%;
  outline: none;
}

.col[disabled] {
  cursor: auto;
}

.row:first-child .col {
  border-top: none;
}

.row:last-child .col {
  border-bottom: none;
}

.col:first-child {
  border-left: none;
}

.col:last-child {
  border-right: none;
}

.col .symbol {
  width: 80%;
}

.symbol-x-enter-active,
.symbol-o-enter-active {
  transition: stroke-dashoffset 0.225s cubic-bezier(0.4,0,0.2,1);
}

.symbol-x-enter {
  stroke-dasharray: 135.764;
  stroke-dashoffset: 135.764;
}

.symbol-x-enter-to {
  stroke-dasharray: 135.764;
  stroke-dashoffset: 0;
}

.symbol-o-enter {
  stroke-dasharray: 301.635;
  stroke-dashoffset: 301.635;
}

.symbol-o-enter-to {
  stroke-dasharray: 301.635;
  stroke-dashoffset: 0;
}
</style>
