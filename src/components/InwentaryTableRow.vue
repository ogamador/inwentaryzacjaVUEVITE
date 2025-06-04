<template>
  <tr>
    <td>{{ row.lp }}</td>

    <td @click="editCell('indeks')">
      <input
        v-if="isEditing('indeks')"
        v-model="localRow.indeks"
        @blur="confirmIndeksEdit"
        @keyup.enter="confirmIndeksEdit"
      />
      <span v-else>{{ row.indeks }}</span>
    </td>

    <td>{{ row.nazwa }}</td>

    <td @click="editCell('ilosc')">
      <input
        v-if="isEditing('ilosc')"
        v-model.number="localRow.ilosc"
        @blur="stopEdit"
        @keyup.enter="stopEdit"
      />
      <span v-else>{{ row.ilosc }}</span>
    </td>

    <td @click="editCell('lokalizacja')">
      <input
        v-if="isEditing('lokalizacja')"
        v-model="localRow.lokalizacja"
        @blur="stopEdit"
        @keyup.enter="stopEdit"
      />
      <span v-else>{{ row.lokalizacja }}</span>
    </td>

    <td>{{ formattedCzas }}</td>

    <td>
      <button @click="$emit('delete', index)">🗑</button>
    </td>
  </tr>
</template>

<script setup>
import { computed, reactive, toRefs, watch } from 'vue'

const props = defineProps({
  row: Object,
  index: Number,
  editingState: Object,
  updateRowName: Function
})
const emit = defineEmits(['edit', 'stop-edit', 'delete'])

const localRow = reactive({
  indeks: props.row.indeks,
  ilosc: props.row.ilosc,
  lokalizacja: props.row.lokalizacja
})

watch(() => props.row, (newRow) => {
  localRow.indeks = newRow.indeks
  localRow.ilosc = newRow.ilosc
  localRow.lokalizacja = newRow.lokalizacja
})

function editCell(field) {
  emit('edit', props.index, field)
}

function stopEdit() {
  emit('stop-edit')
}

function confirmIndeksEdit() {
  props.row.indeks = localRow.indeks
  props.updateRowName(props.row)
  stopEdit()
}

function isEditing(field) {
  return props.editingState.row === props.index && props.editingState.field === field
}

const formattedCzas = computed(() => {
  if (!props.row.czas) return ''
  const date = new Date(props.row.czas)
  return new Intl.DateTimeFormat('pl-PL', {
    day: '2-digit', month: '2-digit', year: 'numeric',
    hour: '2-digit', minute: '2-digit'
  }).format(date)
})
</script>
