<template>
  <div class="chart-container">
    <canvas ref="cpuChart" class="cpu-chart"></canvas>
    <div class="stats">
      <p>Всего запросов: {{ totalRequests }}</p>
      <p>Ошибок: {{ errorCount }} ({{ errorPercentage }}%)</p>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import Chart from 'chart.js/auto'

export default {
  setup() {
    const cpuChart = ref(null)
    const chartInstance = ref(null)
    const totalRequests = ref(0)
    const errorCount = ref(0)
    
    const chartData = {
      labels: [],
      dataPoints: [],
      lastValidValue: 50 
    }

    const errorPercentage = computed(() => {
      return totalRequests.value === 0
        ? 0
        : ((errorCount.value / totalRequests.value) * 100).toFixed(1)
    })

    const initChart = () => {
      if (!cpuChart.value) return

      if (chartInstance.value) {
        chartInstance.value.destroy()
      }

      chartInstance.value = new Chart(cpuChart.value, {
        type: 'line',
        data: {
          labels: chartData.labels,
          datasets: [
            {
              label: 'Загруженность процессора (%)',
              data: chartData.dataPoints,
              borderColor: 'rgba(255, 213, 61, 1)',
              backgroundColor: 'rgba(232, 203, 141, 0.25)',
              fill: true,
              tension: 0.2,
              pointRadius: 2,
            }
          ]
        },
        options: {
          responsive: true,
          animation: false,
          scales: {
            y: {
              beginAtZero: true,
              max: 100,
              ticks: {
                stepSize: 10
              }
            }
          }
        }
      })
    }

    const safeChartUpdate = () => {
      if (!chartInstance.value) return
      
      try {
        const chart = chartInstance.value
        chart.data.labels = [...chartData.labels]
        chart.data.datasets[0].data = [...chartData.dataPoints]
        
        chart.update('none')
      } catch (err) {
        console.error('Ошибка обновления графика:', err)
        initChart()
      }
    }

    const fetchCpuLoad = async () => {
      try {
        const response = await fetch('/api/service/cpu/')

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }
        
        const raw = await response.text()
        const value = parseInt(raw, 10)
        
        let cpuValue
        if (isNaN(value) || value < 0 || value > 100) {
          throw new Error('Invalid CPU value')
        } else {
          cpuValue = value
          chartData.lastValidValue = value
        }

        updateChartData(cpuValue)
        totalRequests.value++
        
      } catch (err) {
        console.error('Ошибка запроса:', err.message)
        errorCount.value++
        totalRequests.value++
        updateChartData(chartData.lastValidValue)
      }
    }

    const updateChartData = (value) => {
      const timestamp = new Date().toLocaleTimeString()
      
      chartData.labels.push(timestamp)
      chartData.dataPoints.push(value)
      
      if (chartData.labels.length > 20) {
        chartData.labels.shift()
        chartData.dataPoints.shift()
      }

      safeChartUpdate()
    }

    onMounted(() => {
      initChart()
      
      fetchCpuLoad()

      const intervalId = setInterval(fetchCpuLoad, 5000)
      
      onBeforeUnmount(() => {
        clearInterval(intervalId)
        if (chartInstance.value) {
          chartInstance.value.destroy()
        }
      })
    })

    return {
      cpuChart,
      totalRequests,
      errorCount,
      errorPercentage
    }
  }
}
</script>

<style scoped>
.chart-container {
  position: relative;
  width: 130%;
  height: 400px;
  margin: 0 auto;
}

.cpu-chart {
  width: 100%;
  height: 100%;
}

.stats {
  margin-top: 1rem;
  font-family: sans-serif;
}
</style>