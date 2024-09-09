<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref, nextTick } from 'vue';

const rows = ref([])
const cols = ref([])
const ROWS_QTD = 100
const COLS_QTD = 10
const container = ref(null)
const table = ref(null)

const range = reactive({
  from: 0,
  to: 0
});

const extraItens = 6
const lineHeight = 50
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

  const itemsOnScreen = Math.ceil(visibleArea / lineHeight) + extraItens;
  const from = Math.min(
    Math.max(0, Math.floor(offHeight / lineHeight) - extraItens / 2),
    ROWS_QTD - 1
  );
  const to = Math.min(from + itemsOnScreen + extraItens / 2, ROWS_QTD - 1);

  range.from = from;
  range.to = to;
}

onMounted(() => {
  for (let i = 0; i < ROWS_QTD; i++) {
    rows.value[i] = { id: i + 1, height: lineHeight }
  }
  for (let j = 0; j < COLS_QTD; j++) {
    cols.value[j] = { id: j + 1, width: colWidth }
  }

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

</script>

<template>
  <div class="table-widget">
    <div>
      <table ref="table" class="overflow-y-auto block">
        <thead>
          <tr>
            <th v-for="col, index in cols" class="border-b border-black" :style="{ width: col.width + 'px' }">H {{ col.id }}</th>
          </tr>
        </thead>
        <tbody ref="container" id="table-rows" class="relative divide-y" :style="{height: ROWS_QTD * lineHeight + 'px'}">
          <tr v-for="i in visibleItems" class="absolute" :style="{
            position: 'absolute',
            top: ((i?.id || 1) - 1) * lineHeight + 'px',
            width: '100%',
            height: lineHeight + 'px'
          }">
            <td v-for="col in cols">{{ `Row ${i.id}, Col ${col.id}` }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped></style>