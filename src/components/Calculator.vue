<template lang="pug">
    .full-width
        .calculator-grid
            .flex.content-center(style="grid-area: display; overflow-x: scroll; overflow-y: hidden; min-height: 5rem;")
                .text-h1.text-right.full-width {{ calculatorDisplay }}
            q-btn(@click="clear" color="grey" style="grid-area: ac;")
                .text-h4.text-bold AC
            q-btn(@click="flipPositiveNegative" color="grey" style="grid-area: plus-minus;")
                .text-h4.text-bold +/-
            q-btn(@click="getPercentage" color="grey" style="grid-area: percent;")
                .text-h4.text-bold %
            q-btn(v-for="btn in buttonOptions" :key="btn.gridArea" :icon="btn.icon" :color="btn.color" :style="`grid-area: ${btn.gridArea};`" @click="onButtonPress(btn)" )
                .text-h4.text-bold {{btn.label}}
            q-btn(@click="evaluate" :style="`grid-area: equals;`" icon="fas fa-equals" color="orange")
</template>
<script>
const buttonOptions = [
  { value: '1', label: '1', gridArea: 'one', type: 'number', color: 'grey' },
  { value: '2', label: '2', gridArea: 'two', type: 'number', color: 'grey' },
  { value: '3', label: '3', gridArea: 'three', type: 'number', color: 'grey' },
  { value: '4', label: '4', gridArea: 'four', type: 'number', color: 'grey' },
  { value: '5', label: '5', gridArea: 'five', type: 'number', color: 'grey' },
  { value: '6', label: '6', gridArea: 'six', type: 'number', color: 'grey' },
  { value: '7', label: '7', gridArea: 'seven', type: 'number', color: 'grey' },
  { value: '8', label: '8', gridArea: 'eight', type: 'number', color: 'grey' },
  { value: '9', label: '9', gridArea: 'nine', type: 'number', color: 'grey' },
  { value: '0', label: '0', gridArea: 'zero', type: 'number', color: 'grey' },
  { value: '.', label: '.', gridArea: 'point', type: 'number', color: 'grey' },
  {
    value: '+',
    icon: 'fas fa-plus',
    gridArea: 'plus',
    type: 'command',
    color: 'orange'
  },
  {
    value: '-',
    icon: 'fas fa-minus',
    gridArea: 'minus',
    type: 'command',
    color: 'orange'
  },
  {
    value: '*',
    icon: 'fas fa-times',
    gridArea: 'times',
    type: 'command',
    color: 'orange'
  },
  {
    value: '%',
    icon: 'fas fa-divide',
    gridArea: 'divide',
    type: 'command',
    color: 'orange'
  }
]

export default {
  data () {
    return {
      buttonOptions,
      calculatorInput: '',
      answer: 0,
      calculatorDisplay: 0,
      displayingAnswer: false,
      history: []
    }
  },
  computed: {
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
    }
  },
  watch: {
    calculatorInput (val) {
      this.calculatorDisplay = this.numbers[this.numbers.length - 1] || 0
    }
  },
  methods: {
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
      this.answer = 0
      this.calculatorInput = ''
    },
    displayAnswer () {
      this.history.push(`${this.calculatorInput} = ${this.answer}`)
      this.displayingAnswer = true
      this.calculatorDisplay = this.answer
      this.calculatorInput = `${this.answer}`
    },
    flipPositiveNegative () {
      if (this.answer < 0) {
        this.answer = Math.abs(this.answer)
      } else this.answer = -Math.abs(this.answer)

      this.displayAnswer()
    },
    getPercentage () {
      this.answer = this.answer / 100
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
        case '/':
          this.answer = this.answer / num
          break
        case '*':
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
  grid-auto-rows: 1fr;
  grid-gap: 0.5rem;
  padding: 0.5rem;
  grid-template-areas:
    'display display display display'
    'ac plus-minus percent divide'
    'seven eight nine times'
    'four five six minus'
    'one two three plus'
    'zero zero point equals';
}
</style>
