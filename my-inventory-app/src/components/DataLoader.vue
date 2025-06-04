<template>
  <div>
    <h2>📂 Wczytaj pliki CSV</h2>

    <div v-if="!ready">
      <label>Plik produktów (indeks, nazwa):</label>
      <input type="file" accept=".csv" @change="handleProduktyUpload" />

      <label>Plik kodów (indeks, kod_kreskowy):</label>
      <input type="file" accept=".csv" @change="handleKodyUpload" />
    </div>

    <p v-if="loading">⏳ Przetwarzanie danych...</p>
    <p v-if="error">❌ Błąd: {{ error }}</p>

    <p v-if="ready">
      ✅ Załadowano {{ produkty.length }} produktów, {{ kody.length }} kodów
      <br />
      <button @click="clearData">🗑️ Wyczyść dane</button>
    </p>

    <slot
      v-if="ready"
      :kodToIndeks="kodToIndeks"
      :indeksToNazwa="indeksToNazwa"
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Papa from 'papaparse'

const produkty = ref([])
const kody = ref([])
const kodToIndeks = ref(new Map())
const indeksToNazwa = ref(new Map())
const loading = ref(false)
const ready = ref(false)
const error = ref(null)

// -- LOCALSTORAGE KEYS --
const STORAGE_KEY_PRODUKTY = 'produktyCSV'
const STORAGE_KEY_KODY = 'kodyCSV'

// -- FILE PARSING --
function parseCSV(file) {
  return new Promise((resolve, reject) => {
    Papa.parse(file, {
      header: true,
      skipEmptyLines: true,
      complete: results => resolve(results.data),
      error: err => reject(err)
    })
  })
}

async function handleProduktyUpload(event) {
  const file = event.target.files[0]
  if (!file) return
  try {
    loading.value = true
    const data = await parseCSV(file)
    produkty.value = data
    localStorage.setItem(STORAGE_KEY_PRODUKTY, JSON.stringify(data))
    rebuildMaps()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

async function handleKodyUpload(event) {
  const file = event.target.files[0]
  if (!file) return
  try {
    loading.value = true
    const data = await parseCSV(file)
    kody.value = data
    localStorage.setItem(STORAGE_KEY_KODY, JSON.stringify(data))
    rebuildMaps()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

// -- MAP BUILDING --
function rebuildMaps() {
  if (produkty.value.length === 0 || kody.value.length === 0) return

  indeksToNazwa.value.clear()
  kodToIndeks.value.clear()

  for (const p of produkty.value) {
    indeksToNazwa.value.set(p.indeks, p.nazwa)
  }

  for (const k of kody.value) {
    kodToIndeks.value.set(k.kod_kreskowy, k.indeks)
  }

  ready.value = true
}

// -- CLEARING STORAGE --
function clearData() {
  localStorage.removeItem(STORAGE_KEY_PRODUKTY)
  localStorage.removeItem(STORAGE_KEY_KODY)
  produkty.value = []
  kody.value = []
  kodToIndeks.value.clear()
  indeksToNazwa.value.clear()
  ready.value = false
}

// -- ON LOAD FROM STORAGE --
onMounted(() => {
  try {
    const savedProdukty = localStorage.getItem(STORAGE_KEY_PRODUKTY)
    const savedKody = localStorage.getItem(STORAGE_KEY_KODY)

    if (savedProdukty && savedKody) {
      produkty.value = JSON.parse(savedProdukty)
      kody.value = JSON.parse(savedKody)
      rebuildMaps()
    }
  } catch (err) {
    error.value = 'Błąd przy odczycie danych z pamięci: ' + err.message
  }
})
</script>

<style scoped>
input {
  margin-bottom: 1rem;
  display: block;
}
button {
  margin-top: 1rem;
  padding: 0.4rem 0.8rem;
  font-size: 0.9rem;
}
</style>
