<script setup>
import { ref, shallowRef, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto'


const weights = ref([])
const weightChartEl = ref(null)
const weightChart = shallowRef(null)
const weightInput = ref(60.0)
const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})
const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })

}
watch(weights, newWeights => {
  const ws = [...newWeights]
  if (weightChart.value) {
    weightChart.value.data.labels = ws.sort((a, b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()).slice(-7)
    weightChart.value.data.datasets[0].data = ws.sort((a, b) => a.date - b.date).map(w => w.weight).slice(-7)
    weightChart.value.update()
    console.log(weightChart.value.data);
    return
  }
  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws.sort((a, b) => a.date - b.date).map(w => new Date(w.date).toLocaleDateString()),
        datasets: [{
          label: 'weight',
          data: ws.sort((a, b) => a.date - b.date).map(w => w.weight),
          backgroundColor: 'rgb(255, 105, 180, 0.2)',
          borderColor: 'rgb(255, 105, 100)',
          borderWidth: 1,
          fill: true
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    })
  })
}, { deep: true })
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current Weight (kg)</small>
    </div>
    <form @submit.prevent="addWeight" class="flex-md-row flex-column">
      <input type="number" v-model="weightInput" step="0.1">
      <input type="submit" value="Add Weight">
    </form>
    <div v-if="weights && weights.length > 0">
      <h2>Last 7 Days</h2>
      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight history</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{ weight.weight }}kg</span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped lang="scss">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "monstserrat", "sans-serif";
}

body {
  background-color: #eee;

  main {
    padding: 1.5rem;

    h1 {
      font-size: 2rem;
      margin-bottom: 2rem;
      text-align: center;
    }

    h2 {
      font-weight: 400;
      color: #888;
      font-size: 1rem;
    }

    .current {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      width: 200px;
      height: 200px;
      text-align: center;
      background-color: white;
      border-radius: 100px;
      border: 5px solid hwb(330 41% 0%);
      margin: 0 auto 2rem;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);

      span {
        display: block;
        font-size: 2rem;
        font-weight: bold;
        margin-bottom: 0.5rem;
      }

      small {
        color: #888;
        font-style: italic;
      }
    }

    form {
      display: flex;
      margin-bottom: 2rem;
      border: 1px solid #aaa;
      overflow: hidden;
      border-radius: 0.5rem;
      transition: 0.2s linear;

      &:focus-within,
      &:hover {
        border-color: hotpink;
        border-width: 2px;
      }

      input {
        appearance: none;
        outline: none;
        border: none;
      }

      input[type="number"] {
        background-color: white;
        flex: 1 1 0%;
        padding: 1rem 1.5rem;
        font-size: 1.25rem;
      }

      input[type="submit"] {
        cursor: pointer;
        background-color: hotpink;
        padding: 0.5rem 1rem;
        color: white;
        font-size: 1.25rem;
        font-weight: bold;
        transition: 0.2s linear;
        border-left: 3px solid transparent;

        &:hover {
          background-color: white;
          color: hotpink;
          border-left-color: hotpink;
        }
      }
    }

    .canvas-box {
      width: 100%;
      background-color: white;
      padding: 1rem;
      border-radius: 0.5rem;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      margin-bottom: 2rem;

      canvas {
        height: 350px;
      }
    }

    .weight-history {
      ul {
        list-style: none;
        padding: 0;
        margin: 0;

        li {
          display: flex;
          cursor: pointer;
          padding: 0.5rem;
          justify-content: space-between;
          align-items: center;

          &:nth-child(even) {
            background-color: #dfdfdfdf;
          }

          &:hover {
            background-color: #f8f8f8f8;
          }

          span {
            display: block;
            font-size: 1.25rem;
            margin-right: 1rem;
            font-weight: bold;
          }

          small {
            color: #888;
            font-style: italic;
          }
        }
      }
    }
  }
  @media(max-width: 767px) {
    input[type="submit"] {
      border: none !important;
    }
  }
}
</style>
