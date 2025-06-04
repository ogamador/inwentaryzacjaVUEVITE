<template>
  <div>
    <video ref="video" width="300" autoplay muted></video>

    <p v-if="skanowanyKod">
      ✅ Zeskanowany kod: <strong>{{ skanowanyKod }}</strong>
      <br />
      Indeks: <strong>{{ dopasowanyIndeks || 'Nie znaleziono' }}</strong>
      <br />
      Nazwa: <strong>{{ dopasowanaNazwa || 'Nie znaleziono' }}</strong>
    </p>

    <h3>📜 Historia skanowania:</h3>
    <ul>
      <li v-for="(entry, index) in historia" :key="index">
        {{ entry.kod }} – {{ entry.indeks }} – {{ entry.nazwa }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import { BrowserMultiFormatReader } from '@zxing/browser'

const props = defineProps({
  kodToIndeks: { type: Object, required: true },
  indeksToNazwa: { type: Object, required: true }
})

const video = ref(null)
const skanowanyKod = ref('')
const dopasowanyIndeks = ref('')
const dopasowanaNazwa = ref('')
const historia = ref([])
let reader

onMounted(() => {
  reader = new BrowserMultiFormatReader()

  reader.decodeFromVideoDevice(null, video.value, (result, err) => {
    if (result) {
      const kod = result.getText()

      if (kod !== skanowanyKod.value) {
        skanowanyKod.value = kod
        reader.reset()  // zatrzymujemy po skanie, można usunąć jeśli chcesz automatycznie dalej skanować
      }
    }
  })
})

watch(skanowanyKod, (kod) => {
  const indeks = props.kodToIndeks.get(kod)
  const nazwa = indeks ? props.indeksToNazwa.get(indeks) : null

  dopasowanyIndeks.value = indeks || ''
  dopasowanaNazwa.value = nazwa || ''

  historia.value.unshift({
    kod,
    indeks: indeks || 'Brak',
    nazwa: nazwa || 'Brak'
  })
})

onUnmounted(() => {
  if (reader) reader.reset()
})
</script>

<style scoped>
ul {
  text-align: left;
  padding: 0;
  list-style: none;
}
li {
  margin-bottom: 0.3rem;
  font-size: 0.95rem;
}
</style>