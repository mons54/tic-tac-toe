<template>
  <div id="app">
    <div class="container">
      <div
        v-if="showResult"
        class="result">
        <div v-if="isWinner">
          Gagné !
        </div>
        <div v-else-if="isWinner === false">
          Perdu !
        </div>
        <div v-else>
          Egalité !
        </div>
        <button v-on:click="init()">
          Rejouer
        </button>
      </div>
      <div
        v-else
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
          {{ col }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'

export default {
  name: 'app',
  data () {
    return {
      isPlayer: null,
      isFinish: false,
      showResult: false,
      isWait: false,
      nbCase: 4,
      values: null,
    }
  },
  methods: {
    isDisabled (row, col) {
      return this.isWait || this.values[row][col] !== null || this.isFinish
    },
    play (row, col) {
      this.values[row][col] = this.isPlayer ? 'x' : 'o'
      Vue.set(this.values[row], col, this.values[row][col])
      this.checkFinish()
    },
    playerPlay (row, col) {
      if (this.isDisabled(row, col))
        return

      this.play(row, col)

      if (this.isFinish) return

      this.isPlayer = !this.isPlayer
      if (!this.isPlayer) {
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

      this.isPlayer = !this.isPlayer
    },
    finish (result = null) {
      this.isWinner = result
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
          return this.finish(this.isPlayer)
      }

      for (let i = 0; i < this.nbCase; i++) {
        values = []
        for (let j = 0; j < this.nbCase; j++) {
          values.push(this.values[j][i])
        }
        if (this.isEgual(values))
          return this.finish(this.isPlayer)
      }

      values = []
      for (let i = 0; i < this.nbCase; i++)
        values.push(this.values[i][i])

      if (this.isEgual(values))
        return this.finish(this.isPlayer)

      values = []
      for (let i = 0; i < this.nbCase; i++)
        values.push(this.values[i][(this.nbCase - 1) - i])

      if (this.isEgual(values))
        return this.finish(this.isPlayer)
        
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
      this.isPlayer = true
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

#app, .container, .row, .col {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: inherit;
}


#app {
  width: 100vw;
  height: 100vh;
  background-color: #4CAF50;
}

.container {
  width: 600px;
  height: 600px;
  flex-direction: column;
}

.result {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: rgba(76, 175, 80, 0.5);
}

.result div {
  font-size: 4rem;
  margin-bottom: 4rem;
  text-transform: uppercase;
  font-weight: 600;
}

.result button {
  border: none;
  padding: 0.8rem 1.6rem;
  font-size: 2rem;
  border-radius: 8px;
  cursor: pointer;
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
  border: 2px solid #000;
  height: 100%;
  font-size: 6rem;
  outline: none;
  color: #000;
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

.col.col-o {
  color: #fff;
}
</style>
