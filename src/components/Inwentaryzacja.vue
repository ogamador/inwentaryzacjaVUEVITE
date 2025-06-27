<template>
  <div>
    <h2>📋 Inwentaryzacja</h2>
    <button @click="clearAll">🗑 Wyczyść wszystkie dane</button>
    <button @click="exportToCSV">💾 Eksportuj do CSV</button>

    <table class="inventory-table">
      <thead>
        <tr>
          <th @click="sortBy('lp')">Lp</th>
          <th @click="sortBy('indeks')">Indeks</th>
          <th @click="sortBy('nazwa')">Nazwa</th>
          <th>Ilość</th>
          <th @click="sortBy('lokalizacja')">Lokalizacja</th>
          <th @click="sortBy('czas')">Czas</th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <InwentaryTableRow
          v-for="(row, i) in sortedRows"
          :key="i"
          :row="row"
          :index="i"
          :editingState="editState"
          :updateRowName="updateNazwa"
          @edit="editCell"
          @stop-edit="stopEdit"
          @delete="(i) => rows.splice(i, 1)"
        />

        <tr>
          <td>{{ rows.length + 1 }}</td>
          <td>
            <input
              ref="indeksInput"
              v-model="current.indeks"
              @keyup.enter="onIndeksEntered"
              @focus="onInputFocus"
              autofocus
              type="text"
              inputmode="text"
              autocomplete="off"
              autocorrect="off"
              autocapitalize="off"
            />
          </td>
          <td>{{ current.nazwa }}</td>
          <td><input
            v-model.number="current.ilosc"
            @keyup.enter="onIloscEntered"
            @focus="e => e.target.select()"
            type="number"
            inputmode="numeric"
            pattern="[0-9]*"/></td>
          <td><input v-model="current.lokalizacja" @keyup.enter="onLokalizacjaEntered" /></td>
          <td>{{ formatDate(current.czas) }}</td>
          <td><button @click="onLokalizacjaEntered">✅</button></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted, onUnmounted } from 'vue'
import { saveAs } from 'file-saver'
import InwentaryTableRow from './InwentaryTableRow.vue'

const { kodToIndeks, indeksToNazwa } = defineProps(['kodToIndeks', 'indeksToNazwa'])

let lastClipboard = ''
let clipboardInterval

const STORAGE_KEY = 'inwentaryzacjaRows'
const rows = ref([])
const current = reactive({
  indeks: '',
  nazwa: '',
  ilosc: 1,
  lokalizacja: '',
  czas: ''
})
const indeksInput = ref(null)

const sortKey = ref(null)
const sortDirection = ref(1)

const sortedRows = computed(() => {
  if (!sortKey.value) return rows.value
  return [...rows.value].sort((a, b) => {
    const valA = a[sortKey.value] || ''
    const valB = b[sortKey.value] || ''
    if (sortKey.value === 'lp' || sortKey.value === 'ilosc') {
      return (valA - valB) * sortDirection.value
    }
    if (sortKey.value === 'czas') {
      return (new Date(valA) - new Date(valB)) * sortDirection.value
    }
    return valA.localeCompare(valB, 'pl', { sensitivity: 'base' }) * sortDirection.value
  })
})

function sortBy(key) {
  if (sortKey.value === key) {
    sortDirection.value *= -1
  } else {
    sortKey.value = key
    sortDirection.value = 1
  }
}

onMounted(() => {
  // 🔹 Focusuj pole indeksu na start
  indeksInput.value?.focus()

  // 🔹 Wczytaj dane z localStorage
  const saved = localStorage.getItem(STORAGE_KEY)
  if (saved) {
    try {
      rows.value = JSON.parse(saved)
    } catch (err) {
      console.warn('Błąd odczytu localStorage:', err)
    }
  }

  // 🔹 Utrzymuj focus gdy znika (dla skanera)
  window.addEventListener('keydown', (e) => {
    const tag = document.activeElement?.tagName
    if (tag !== 'INPUT') {
      indeksInput.value?.focus()
    }
  })

  // 🔹 Interval: nasłuchiwanie schowka (clipboard API)
  clipboardInterval = setInterval(async () => {
    try {
      const text = await navigator.clipboard.readText()
      if (text && text !== lastClipboard) {
        lastClipboard = text
        if (text.length === 13) {
          // kod EAN → indeks
          current.value.indeks = text
          onIndeksEntered()
        } else {
          // lokalizacja
          current.value.lokalizacja = text
          onLokalizacjaEntered()
        }
      }
    } catch (err) {
      // brak uprawnień do clipboarda – zignoruj
    }
  }, 1000)
})

onUnmounted(() => {
  clearInterval(clipboardInterval)
})

watch(rows, (newRows) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(newRows))
}, { deep: true })

function formatDate(isoString) {
  if (!isoString) return ''
  const date = new Date(isoString)
  return new Intl.DateTimeFormat('pl-PL', {
    day: '2-digit', month: '2-digit', year: 'numeric',
    hour: '2-digit', minute: '2-digit'
  }).format(date)
}

watch(() => current.indeks, (val) => {
  updateNazwa(current)
})

function updateNazwa(row) {
  let indeks = row.indeks
  let nazwa = indeksToNazwa.get(indeks)

  if (!nazwa && kodToIndeks.has(indeks)) {
    indeks = kodToIndeks.get(indeks)
    nazwa = indeksToNazwa.get(indeks)
    row.nazwa = `${indeks} ${nazwa || ''}`
  } else {
    row.nazwa = nazwa || ''
  }
}

function onInputFocus(e) {
  e.target.select()
}

function onIndeksEntered() {
  current.ilosc = 1
  setTimeout(() => document.querySelector('input[v-model="current.ilosc"]')?.focus(), 0)
}

function onIloscEntered() {
  setTimeout(() => document.querySelector('input[v-model="current.lokalizacja"]')?.focus(), 0)
}

function onLokalizacjaEntered() {
  if (!current.indeks || !current.nazwa || current.ilosc <= 0) return

  const now = new Date().toISOString()
  const newRow = {
    lp: rows.value.length + 1,
    indeks: current.indeks,
    nazwa: current.nazwa,
    ilosc: current.ilosc,
    lokalizacja: current.lokalizacja,
    czas: now
  }
  rows.value.push(newRow)
  const lastLocation = current.lokalizacja
  current.indeks = ''
  current.nazwa = ''
  current.ilosc = 1
  current.lokalizacja = lastLocation
  current.czas = ''
  setTimeout(() => indeksInput.value?.focus(), 0)
}

function exportToCSV() {
  if (rows.value.length === 0) {
    alert('Brak danych do eksportu.')
    return
  }
  const headers = ['Zeskanowany Indeks', 'Symbol', 'Nazwa', 'Ilość', 'Lp', 'Lokalizacja', 'Czas']
  const csvContent = [
    headers.join(';'),
    ...rows.value.map(row => {
      const zeskanowanyIndeks = `="${row.indeks}"`
      let symbol = row.indeks
      let nazwa = row.nazwa

      if (kodToIndeks.has(row.indeks)) {
        symbol = kodToIndeks.get(row.indeks)
        nazwa = indeksToNazwa.get(symbol) || row.nazwa
      }

      const symbolFormatted = `="${symbol}"`

      return [
        zeskanowanyIndeks,
        symbolFormatted,
        nazwa,
        row.ilosc,
        row.lp,
        row.lokalizacja,
        formatDate(row.czas)
      ].join(';')
    })
  ].join('\n')

  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const fileName = `inwentaryzacja_${new Date().toISOString().slice(0, 10)}.csv`
  saveAs(blob, fileName)
}

const editState = ref({ row: null, field: null })
function editCell(rowIndex, field) {
  editState.value = { row: rowIndex, field }
}
function stopEdit() {
  editState.value = { row: null, field: null }
}
function clearAll() {
  if (confirm('Czy na pewno chcesz usunąć wszystkie dane inwentaryzacji?')) {
    rows.value = []
    localStorage.removeItem(STORAGE_KEY)
  }
}
</script>