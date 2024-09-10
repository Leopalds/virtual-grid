<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref, nextTick } from 'vue';
import throttle from 'lodash/throttle'

const rows = ref([])
const cols = ref([])
const ROWS_QTD = 100_000
const COLS_QTD = 10
const container = ref(null)
const table = ref(null)

const range = reactive({
  from: 0,
  to: 0
});

const extraItens = 3
const minHeight = 50
const colWidth = 100

const rowHeights = ref([])
const totalTableHeight = ref(0)


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
    totalHeight += rowHeights.value[i] || minHeight
    if (totalHeight >= offHeight) {
      from = i;
      break;
    }
  }

  totalHeight = 0;
  for (let i = from; i < ROWS_QTD; i++) {
    totalHeight += rowHeights.value[i] || minHeight
    if (totalHeight >= visibleArea) {
      to = i;
      break;
    }
  }

  range.from = Math.max(0, from - extraItens)
  range.to = Math.min(ROWS_QTD - 1, to + extraItens)
}

const throttledRenderRows = throttle(renderRows, 100)

const updateRowHeight = (index, element) => {
  const realHeight = element.getBoundingClientRect().height
  
  if(realHeight !== rowHeights.value[index]){
    const heightDifference = realHeight - rowHeights.value[index] || minHeight;
    rowHeights.value[index] = realHeight;
    totalTableHeight.value += heightDifference;

    throttledRenderRows();
  }
}

const onRowRendered = (index, element) => {
  if (rowHeights.value[index] === minHeight) {
    updateRowHeight(index, element);
  }
};

onMounted(() => {
  for (let i = 0; i < ROWS_QTD; i++) {
    const height = Math.floor(Math.random() * (200 - 50 + 1)) + 50
    rows.value[i] = { id: i + 1, height }
    rowHeights.value[i] = minHeight
  }

  totalTableHeight.value = rowHeights.value.length * minHeight;

  for (let j = 0; j < COLS_QTD; j++) {
    cols.value[j] = { id: j + 1, width: colWidth }
  }

  nextTick(() => throttledRenderRows())
  //Throttle
  table.value.addEventListener("scroll", throttledRenderRows)
})

onUnmounted(() => {
  table.value.removeEventListener("scroll", throttledRenderRows)
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
  for (let i = 0; i < index; i++) {
    offset += rowHeights.value[i] || minHeight;
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
          <tr v-for="i in visibleItems"
            :style="{
              position: 'absolute',
              top: getTopOffset(i?.id - 1) + 'px',
              width: '100%',
              height: i?.height + 'px'
            }"
            :ref="el => $nextTick(() => onRowRendered(i?.id - 1, el))"
          >
            <td v-for="col in cols">{{ `Row ${i.id}, Col ${col.id}` }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped></style>