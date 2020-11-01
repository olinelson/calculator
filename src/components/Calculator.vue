<template lang="pug">
    .full-width
        .calculator-grid
            .calculator-display.row.justify-end.content-center.q-pa-md
              .text-h1.text-white {{ calculatorDisplay }}
            q-btn.no-border-radius(@click="clear" color="primary" style="grid-area: ac;")
                .text-h4.text-bold {{clearButtonLabel}}
            q-btn.no-border-radius(@click="flipPositiveNegative" color="primary" style="grid-area: plus-minus;")
                .text-h4.text-bold +/-
            q-btn.no-border-radius(@click="getPercentage" color="primary" style="grid-area: percent;")
                .text-h4.text-bold %
            q-btn.no-border-radius(v-for="btn in buttonOptions" :key="btn.gridArea" :icon="btn.icon" :color="btn.color" :style="`grid-area: ${btn.gridArea};`" @click="onButtonPress(btn)" )
                .text-h4.text-bold {{btn.label}}
            q-btn.no-border-radius(@click="evaluate" :style="`grid-area: equals;`" icon="fas fa-equals" color="accent")
        keypress(key-event="keyup" @success="onKeyPress")

</template>
<script>
import { db } from 'boot/localbase'
import { uid } from 'uid'
import { mapState, mapActions } from 'vuex'
import buttonOptions from '../data/buttonOptions'
import keypress from 'vue-keypress'
import { allowedNumberInputs, allowedSymbolInputs } from '../data/allowedInputs'

export default {
  components: {
    keypress
  },
  data () {
    return {
      buttonOptions,
      calculatorInput: '',
      answer: null,
      calculatorDisplay: 0,
      displayingAnswer: false,
      allowedNumberInputs,
      allowedSymbolInputs
    }
  },
  computed: {
    ...mapState('history', ['history']),
    numbers () {
      return this.calculatorInput
        .split(' ')
        .filter(i => parseFloat(i) && i.length)
        .map(i => parseFloat(i))
    },
    commands () {
      return this.calculatorInput
        .split(' ')
        .filter(i => !parseFloat(i) && i.length)
    },
    clearButtonLabel () {
      if (this.calculatorInput.length && !this.displayingAnswer) return 'C'
      return 'AC'
    }
  },
  watch: {
    calculatorInput (val) {
      this.calculatorDisplay = this.numbers[this.numbers.length - 1] || 0
    }
  },
  methods: {
    ...mapActions('history', ['setHistory']),
    onKeyPress (e) {
      const key = e.event.key
      if (allowedNumberInputs.includes(key)) {
        return this.onNumberPress(key)
      } else if (allowedSymbolInputs.includes(key)) {
        return this.onCommandPress(key)
      } else if (key === '=' || key === 'Enter') {
        return this.evaluate()
      } else if (key === 'c') {
        return this.clear()
      }
    },
    onButtonPress (btn) {
      if (btn.type === 'number') this.onNumberPress(btn.value)
      else this.onCommandPress(btn.value)
    },
    onNumberPress (num) {
      if (this.displayingAnswer) {
        this.calculatorInput = num
        this.displayingAnswer = false
      } else this.calculatorInput += num
    },
    onCommandPress (command) {
      if (this.displayingAnswer) this.displayingAnswer = false
      this.calculatorInput += ` ${command} `
    },
    clear () {
      this.answer = null
      this.calculatorInput = ''
    },
    async saveAnswerToHistory (calculatorInput, answer) {
      try {
        const id = uid()
        await db.collection('history').add({
          id,
          createdAt: Date.now(),
          sum: `${calculatorInput} = ${answer}`
        })
        const newHistoryItem = await db
          .collection('history')
          .doc({ id })
          .get()
        this.setHistory([...this.history, newHistoryItem])
      } catch (error) {
        this.$q.notify({
          message: 'Error saving history',
          color: 'negative',
          icon: 'fas fa-times'
        })
      }
    },
    displayAnswer () {
      this.displayingAnswer = true
      this.saveAnswerToHistory(this.calculatorInput, this.answer)
      this.calculatorDisplay = this.answer
      this.calculatorInput = `${this.answer}`
    },
    getAnswerOrLastNumber () {
      if (this.answer) {
        return this.answer
      }
      return this.numbers[this.numbers.length - 1]
    },
    flipPositiveNegative () {
      const num = this.getAnswerOrLastNumber()

      if (num < 0) {
        this.answer = Math.abs(num)
      } else this.answer = -Math.abs(num)

      this.displayAnswer()
    },
    getPercentage () {
      const num = this.getAnswerOrLastNumber()

      this.answer = num / 100
      this.displayAnswer()
    },
    evaluate () {
      this.answer = this.numbers[0]
      for (let i = 1; i < this.numbers.length; i += 1) {
        this.buildAnswer(this.commands[i - 1], this.numbers[i])
      }
      this.displayAnswer()
    },
    buildAnswer (command, num) {
      switch (command) {
        case '+':
          this.answer = this.answer + num
          break
        case '-':
          this.answer = this.answer - num
          break
        case '÷':
          this.answer = this.answer / num
          break
        case 'x':
          this.answer = this.answer * num
          break

        default:
          break
      }
    }
  }
}
</script>

<style>
.calculator-grid {
  display: grid;
  height: 90vh;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  grid-auto-rows: minmax(1fr, auto);
  grid-gap: 0.1rem;
  grid-template-areas:
    'display display display display'
    'ac plus-minus percent divide'
    'seven eight nine times'
    'four five six minus'
    'one two three plus'
    'zero zero point equals';
}
.calculator-display {
  grid-area: display;
  overflow-x: scroll;
  overflow-y: hidden;
  min-height: 5rem;
}
</style>
