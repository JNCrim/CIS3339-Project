
<template>
  <div>
    <canvas class="p-10" ref="attendanceChart"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { Chart, registerables, Colors } from 'chart.js'

Chart.register(...registerables)
Chart.register(Colors)

// Composition API: defineProps() replaces the Options API
// props declaration and preserves the existing prop names.
const props = defineProps({
  label: {
    type: Array
  },
  chartData: {
    type: Array
  }
})

// Composition API: ref() replaces this.$refs.attendanceChart.
const attendanceChart = ref(null)

// Retain the Chart.js instance for updates and cleanup.
let chartInstance = null

// Composition API: onMounted() replaces mounted().
// Create the chart after its canvas element is available.
onMounted(() => {
  chartInstance = new Chart(attendanceChart.value, {
    type: 'bar',
    data: {
      labels: props.label || [],
      datasets: [
        {
          borderWidth: 1,
          data: props.chartData || []
        }
      ]
    },
    options: {
      scales: {
        y: {
          ticks: {
            stepSize: 1
          }
        },
        x: {
          grid: {
            display: false
          }
        }
      },
      plugins: {
        legend: {
          display: false
        },
        title: {
          display: true,
          text: 'Attendance Chart'
        }
      },
      responsive: true,
      maintainAspectRatio: true
    }
  })
})

// Update the existing chart when dashboard data changes.
watch(
  () => [props.label, props.chartData],
  () => {
    if (!chartInstance) return

    chartInstance.data.labels = props.label || []
    chartInstance.data.datasets[0].data = props.chartData || []
    chartInstance.update()
  },
  { deep: true }
)

// Composition API: onUnmounted() cleans up the Chart.js
// instance when the component is removed.
onUnmounted(() => {
  if (chartInstance) {
    chartInstance.destroy()
    chartInstance = null
  }
})
</script>
