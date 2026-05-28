<template>
  <div class="convertisseur">
    <h2>Taux de change</h2>

    <!-- Input XPF maître -->
    <div class="devise-row master">
      <img :src="getFlagUrl('XPF')" alt="XPF" class="flag" />
      <div class="devise-info">
        <span class="devise-nom">Franc Pacifique</span>
        <span class="devise-code">XPF</span>
      </div>
      <input type="number" v-model.number="xpf" min="0" class="devise-input" />
    </div>

    <div v-if="loading" class="loading">Chargement...</div>
    <p v-else-if="error">Erreur : {{ error }}</p>

    <!-- Une ligne par devise -->
    <div
      v-else
      v-for="(rate, currency) in data.conversion_rates"
      :key="currency"
      class="devise-row"
    >
      <img :src="getFlagUrl(currency)" alt="" class="flag" />
      <div class="devise-info">
        <span class="devise-code">{{ currency }}</span>
      </div>
      <input type="number" :value="convertViaXPF(rate)" readonly class="devise-input" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const data = ref(null)
const loading = ref(true)
const error = ref(null)
const xpf = ref(1000)
let intervalId = null

// FlagPedia utilise des codes pays ISO 2 lettres
// On mappe les codes devises → codes pays
const currencyToCountry = {
  USD: 'us', EUR: 'eu', GBP: 'gb', JPY: 'jp', AUD: 'au',
  CAD: 'ca', CHF: 'ch', CNY: 'cn', NZD: 'nz', XPF: 'pf',
  SGD: 'sg', HKD: 'hk', NOK: 'no', SEK: 'se', DKK: 'dk',
  MXN: 'mx', BRL: 'br', ZAR: 'za', INR: 'in', KRW: 'kr',
  // Ajoute d'autres selon tes besoins
}

function getFlagUrl (currency) {
  const code = currencyToCountry[currency]
  if (!code) return 'https://flagcdn.com/w40/un.png'  // drapeau ONU par défaut
  return `https://flagcdn.com/w40/${code}.png`
}

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

onMounted(() => {
  fetchData()
  intervalId = setInterval(fetchData, 3600000)
})

onUnmounted(() => {
  clearInterval(intervalId)
})
</script>
