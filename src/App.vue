<template>
  <div class="app">
    <h1>📷 Inwentaryzacja roślin</h1>

    <DataLoader>
      <template #default="{ kodToIndeks, indeksToNazwa }">
        <Inwentaryzacja :kodToIndeks="kodToIndeks" :indeksToNazwa="indeksToNazwa" />
      </template>
    </DataLoader>
  </div>
</template>

<script setup>
import DataLoader from './components/DataLoader.vue'
import Inwentaryzacja from './components/Inwentaryzacja.vue'
</script>

<style>
.app {
  padding: 2rem;
  font-family: sans-serif;
}
</style>
