<template>
  <div>
    <label for="outcomes">Количество исходов:</label>
    <input type="number" v-model.number="numberOfOutcomes" @input="updateOutcomes" min="1">

    <div v-for="n in numberOfOutcomes" :key="n">
      <label :for="'outcome' + n">Исход {{ n - 1 }}:</label>
      <input type="number" v-model.number="outcomes[n - 1]" min="0" max="1" step="0.01">
    </div>

    <div>
      <h3>Вероятности исходов:</h3>
      <ul>
        <li v-for="(probability, index) in outcomes" :key="index">
          Исход {{ index }}: {{ probability }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      numberOfOutcomes: 1,
      outcomes: [0]
    }
  },
  methods: {
    updateOutcomes() {
      if (this.numberOfOutcomes > this.outcomes.length) {
        for (let i = this.outcomes.length; i < this.numberOfOutcomes; i++) {
          this.outcomes.push(0)
        }
      } else {
        this.outcomes = this.outcomes.slice(0, this.numberOfOutcomes)
      }
    }
  }
}
</script>

<style scoped>
/* Добавьте ваши стили здесь */
</style>
