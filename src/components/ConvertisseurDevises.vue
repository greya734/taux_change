<template>
  <div>

    <p v-if="loading">Chargement...</p>

    <p v-else-if="error">Erreur : {{ error }}</p>

    <div v-else-if="data">
      <h2>Taux de change (base : {{ data.base_code }})</h2>
      <label>XPF</label>
      <input type="number" v-model.number="xpf" min="0" />

      <div v-for="(rate, currency) in data.conversion_rates" :key="currency">
        <label>{{ currency }}</label>
        <input type="number" :value="convertViaXPF(rate)" readonly />
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const data = ref(null)
const loading = ref(true)
const error = ref(null)
const xpf = ref(1000)

const xpfEnUsd = computed(() => {
  if (!data.value) return 1
  return 1 / data.value.conversion_rates.XPF
})

function convertViaXPF (rate) {
  const usd = xpf.value * xpfEnUsd.value
  return (usd * rate).toFixed(2)
}

async function fetchData () {
  try {
    const response = await fetch('https://v6.exchangerate-api.com/v6/bfbbeab7192d7bbefeabf7b9/latest/USD')
    if (!response.ok) throw new Error(`Erreur HTTP: ${response.status}`)
    data.value = await response.json()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
</script>
