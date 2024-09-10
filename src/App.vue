<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref, nextTick } from 'vue';

const rows = ref([])
const rowHeights = ref([])
const totalTableHeight = ref(0)
const cols = ref([])
const ROWS_QTD = 100_000
const COLS_QTD = 10
const container = ref(null)
const table = ref(null)

const range = reactive({
  from: 0,
  to: 0
});

const extraItens = 6
const minHeight = 50
const colWidth = 100

const renderRows = () => {
  const windowHeight = table.value.clientHeight;
  const scrollTop = table.value.scrollTop;
  const offsetTop = container.value?.offsetTop || 0;
  const compHeight = container.value?.clientHeight || 0;
  const distanceToTop = offsetTop - scrollTop;
  const offHeight = Math.max(0, scrollTop - offsetTop);
  const visibleArea = Math.max(
    0,
    Math.min(
      windowHeight,
      windowHeight - distanceToTop,
      -(-distanceToTop - compHeight)
    )
  );

  let totalHeight = 0;
  let from = 0;
  let to = 0;
  
  for (let i = 0; i < ROWS_QTD; i++) {
    totalHeight += rowHeights.value[i];
    if (totalHeight >= offHeight) {
      from = i;
      break;
    }
  }

  totalHeight = 0;
  for (let i = from; i < ROWS_QTD; i++) {
    totalHeight += rowHeights.value[i];
    if (totalHeight >= visibleArea) {
      to = i;
      break;
    }
  }

  range.from = from;
  range.to = to;
}

onMounted(() => {
  for (let i = 0; i < ROWS_QTD; i++) {
    const height = Math.floor(Math.random() * (200 - 50 + 1)) + 50
    rows.value[i] = { id: i + 1, height }
    rowHeights.value[i] = height
  }
  for (let j = 0; j < COLS_QTD; j++) {
    cols.value[j] = { id: j + 1, width: colWidth }
  }
  totalTableHeight.value = rowHeights.value.reduce((acc, height) => acc + height, 0);
  nextTick(() => renderRows())
  table.value.addEventListener("scroll", renderRows)
})

onUnmounted(() => {
  table.value.removeEventListener("scroll", renderRows)
})


const visibleItems = computed(() => {
  const items = []
  for (let i = range.from; i <= range.to; i++) {
    items.push(rows.value[i]);
  }
  return items;
})

const getTopOffset = (index) => {
  let offset = 0
  for(let i = 0; i < index; i++){
    offset += rowHeights.value[i]
  }
  return offset
}

</script>

<template>
  <div class="table-widget">
    <div>
      <table ref="table" class="overflow-y-auto block">
        <thead class="sticky top-0 bg-white z-10">
          <tr>
            <th v-for="col, index in cols" class="border-b border-black bg-white" :style="{ width: col.width + 'px' }">H {{ col.id }}</th>
          </tr>
        </thead>
        <tbody ref="container" id="table-rows" class="relative divide-y" :style="{height: totalTableHeight + 'px'}">
          <tr v-for="i in visibleItems" class="absolute" :style="{
            position: 'absolute',
            top: getTopOffset(i?.id - 1) + 'px',
            width: '100%',
            height: i?.height + 'px'
          }">
            <td v-for="col in cols">{{ `Row ${i.id}, Col ${col.id}` }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped></style>