<template>
  <div class="w-1/2">
    <canvas class="p-10" ref="zipChart"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { Chart, registerables } from 'chart.js'

// Register the Chart.js components needed to render
// the doughnut chart, as in the original Options API.
Chart.register(...registerables)

// Composition API: defineProps() replaces the Options API
// props declaration. The names remain unchanged so
// home.vue can continue passing data to this component.
const props = defineProps({
  label: {
    type: Array
  },
  chartData: {
    type: Array
  }
})

// Composition API: ref() replaces this.$refs.zipChart.
// The canvas element becomes available after mounting.
const zipChart = ref(null)

// Keep a reference to the Chart.js instance so we can
// update it and clean it up when the component unmounts.
let chartInstance = null

// Composition API: onMounted() replaces mounted().
// Create the doughnut chart once the canvas is available.
onMounted(() => {
  chartInstance = new Chart(zipChart.value, {
    type: 'doughnut',
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
      plugins: {
        legend: {
          display: false
        },
        title: {
          display: true,
          text: 'Clients by Zip Code'
        }
      },
      responsive: true,
      maintainAspectRatio: true
    }
  })
})

// Watch for changes to the ZIP code data received
// from home.vue. Update the existing chart when
// the API request finishes or the data changes.
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

// Composition API: onUnmounted() replaces the need
// for manual chart cleanup in the Options API.
// Destroy the chart when navigating away.
onUnmounted(() => {
  if (chartInstance) {
    chartInstance.destroy()
    chartInstance = null
  }
})
</script>

