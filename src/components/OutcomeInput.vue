<template>
  <div class="outcome-input">
    <h1>Управление серийностью в случайности</h1>
    <div class="input-group">
      <label>Количество возможных исходов:</label>
      <input type="number" v-model.number="numOutcomes" @input="updateOutcomes" />
    </div>
    <div v-for="(probability, index) in probabilities" :key="index" class="input-group">
      <label>Вероятность исхода {{ index }}:</label>
      <input type="number" v-model.number="probabilities[index]" @input="updateProbabilities" />
      <span class="fraction">{{ fractions[index].toFixed(3) }}</span>
    </div>
    <div v-for="(divider, index) in dividers" :key="index" class="input-group">
      <label>Делитель для исхода {{ index }}:</label>
      <input type="number" v-model.number="dividers[index]" />
    </div>
    <div class="input-group">
      <label>Количество испытаний:</label>
      <input type="number" v-model.number="numberOfTrials" />
    </div>
    <button @click="startTrials">Начать</button>
    <button @click="startTrialsWithDividers">Начать с делителями</button>

    <h2>Результаты испытаний:</h2>
    <div class="results">{{ results.join(' ') }}</div>

    <h2>Результаты испытаний с делителями:</h2>
    <div class="results">{{ resultsWithDividers.join(' ') }}</div>

    <h2>Статистика:</h2>
    <div v-for="(stat, index) in statistics" :key="index" class="statistics">
      <h3>Исход {{ index }}</h3>
      <div>Средняя длина серии: {{ stat.averageStreak.toFixed(2) }}</div>
      <div>Максимальная длина серии: {{ stat.maxStreak }}</div>
      <div>Среднее количество появлений: {{ stat.averageOccurrences.toFixed(2) }}</div>
    </div>

    <h2>Статистика с делителями:</h2>
    <div v-for="(stat, index) in statisticsWithDividers" :key="index" class="statistics">
      <h3>Исход {{ index }}</h3>
      <div>Средняя длина серии: {{ stat.averageStreak.toFixed(2) }}</div>
      <div>Максимальная длина серии: {{ stat.maxStreak }}</div>
      <div>Среднее количество появлений: {{ stat.averageOccurrences.toFixed(2) }}</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      numOutcomes: 3,
      probabilities: [2, 1, 2], // Initial probabilities as integers
      dividers: [2, 3, 1.5],
      fractions: [0.4, 0.2, 0.4], // Initial fractions
      numberOfTrials: 100,
      results: [],
      resultsWithDividers: [],
      statistics: [],
      statisticsWithDividers: []
    };
  },
  methods: {
    updateOutcomes() {
      this.probabilities = Array(this.numOutcomes).fill(1);
      this.dividers = Array(this.numOutcomes).fill(1);
      this.updateProbabilities();
    },
    updateProbabilities() {
      const sum = this.probabilities.reduce((acc, prob) => acc + prob, 0);
      this.fractions = this.probabilities.map(prob => prob / sum);
    },
    startTrials() {
      this.results = [];
      for (let i = 0; i < this.numberOfTrials; i++) {
        const outcome = this.getRandomOutcome(this.fractions);
        this.results.push(outcome);
      }
      this.calculateStatistics();
    },
    startTrialsWithDividers() {
      this.resultsWithDividers = [];
      let currentFractions = [...this.fractions];
      for (let i = 0; i < this.numberOfTrials; i++) {
        const outcome = this.getRandomOutcome(currentFractions);
        this.resultsWithDividers.push(outcome);
        currentFractions = this.calculateNewProbabilities(outcome);
      }
      this.calculateStatisticsWithDividers();
    },
    getRandomOutcome(probabilities) {
      const rand = Math.random();
      let sum = 0;
      for (let i = 0; i < probabilities.length; i++) {
        sum += probabilities[i];
        if (rand < sum) return i;
      }
      return probabilities.length - 1;
    },
    calculateNewProbabilities(outcome) {
      const newProbabilities = [...this.fractions];
      newProbabilities[outcome] = this.fractions[outcome] / this.dividers[outcome];

      const sumOthers = newProbabilities.reduce((acc, prob, index) => {
        return index !== outcome ? acc + prob : acc;
      }, 0);

      const scalingFactor = (1 - newProbabilities[outcome]) / sumOthers;

      return newProbabilities.map((prob, index) => {
        return index !== outcome ? prob * scalingFactor : prob;
      });
    },
    calculateStatistics() {
      const stats = Array(this.numOutcomes).fill(null).map(() => ({
        streaks: [],
        totalOccurrences: 0,
      }));

      let currentStreak = 1;
      for (let i = 0; i < this.results.length; i++) {
        const outcome = this.results[i];
        stats[outcome].totalOccurrences += 1;
        if (i > 0 && this.results[i] === this.results[i - 1]) {
          currentStreak += 1;
        } else {
          if (i > 0) {
            stats[this.results[i - 1]].streaks.push(currentStreak);
          }
          currentStreak = 1;
        }
      }
      stats[this.results[this.results.length - 1]].streaks.push(currentStreak);

      this.statistics = stats.map(stat => ({
        averageStreak: stat.streaks.reduce((a, b) => a + b, 0) / stat.streaks.length,
        maxStreak: Math.max(...stat.streaks),
        averageOccurrences: stat.totalOccurrences / this.numberOfTrials,
      }));
    },
    calculateStatisticsWithDividers() {
      const stats = Array(this.numOutcomes).fill(null).map(() => ({
        streaks: [],
        totalOccurrences: 0,
      }));

      let currentStreak = 1;
      for (let i = 0; i < this.resultsWithDividers.length; i++) {
        const outcome = this.resultsWithDividers[i];
        stats[outcome].totalOccurrences += 1;
        if (i > 0 && this.resultsWithDividers[i] === this.resultsWithDividers[i - 1]) {
          currentStreak += 1;
        } else {
          if (i > 0) {
            stats[this.resultsWithDividers[i - 1]].streaks.push(currentStreak);
          }
          currentStreak = 1;
        }
      }
      stats[this.resultsWithDividers[this.resultsWithDividers.length - 1]].streaks.push(currentStreak);

      this.statisticsWithDividers = stats.map(stat => ({
        averageStreak: stat.streaks.reduce((a, b) => a + b, 0) / stat.streaks.length,
        maxStreak: Math.max(...stat.streaks),
        averageOccurrences: stat.totalOccurrences / this.numberOfTrials,
      }));
    }
  },
  mounted() {
    this.updateOutcomes();
  }
};
</script>

<style scoped lang="scss">
.outcome-input {
  font-family: Arial, sans-serif;

  h1 {
    font-size: 20px;
    margin: 0;
  }

  h2 {
    font-size: 15px;
    margin-top: 20px;
  }

  h3 {
    font-size: 15px;
  }

  .input-group {
    display: flex;
    align-items: center;
    margin-bottom: 10px;

    label {
      margin-right: 10px;
      min-width: 200px;
      text-align: right;
    }

    input {
      margin-right: 10px;
    }

    .fraction {
      margin-left: 10px;
      font-size: 14px;
    }
  }

  .results, .statistics {
    margin-bottom: 2px;

    div {
      margin: 2px 0;
    }
  }
}
</style>
