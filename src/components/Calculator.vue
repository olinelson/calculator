<template lang="pug">
    div
        h1 Calculator
        h4 {{calculatorDisplay}}
        q-input(v-model="calculatorInput")
        q-btn(@click="evaluate" color="primary" label="=")
        q-btn(@click="flipPositiveNegative" color="primary" label="+/-")
        q-btn(@click="getPercentage" color="primary" label="%")
        q-btn(@click="clear" color="primary" label="AC")
        h4 {{answer}}
</template>
<script>
export default {
  data () {
    return {
      calculatorInput: '',
      answer: 0,
      calculatorDisplay: 0
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
      this.calculatorDisplay = this.numbers[this.numbers.length - 1]
    }
  },
  methods: {
    clear () {
      this.answer = 0
      this.displayAnswer()
    },
    displayAnswer () {
      this.calculatorDisplay = this.answer
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
