<template>
  <div>
    <!-- The table itself. Tailwind classes handle the spacing and borders. -->
    <table class="min-w-full divide-y divide-gray-200">

      <!-- HEADER ROW: one clickable header cell per column you pass in. -->
      <thead class="bg-gray-50">
        <tr>
          <!-- v-for loops over the columns prop and makes one <th> for each. -->
          <!-- Clicking (or pressing Enter on) a header calls toggleSort for that column. -->
          <th
            v-for="col in columns"
            :key="col.field"
            class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer select-none"
            tabindex="0"
            @click="toggleSort(col.field)"
            @keydown.enter="toggleSort(col.field)"
          >
            {{ col.label }}
            <!-- Shows an up or down arrow only on the column currently being sorted. -->
            <span v-if="sortField === col.field">
              {{ sortDir === 'asc' ? '▲' : '▼' }}
            </span>
          </th>
        </tr>
      </thead>

      <tbody class="bg-white divide-y divide-gray-200">

        <!-- EMPTY STATE: if there are no rows at all, show one "No results" row. -->
        <!-- colspan makes that one cell stretch across every column. -->
        <tr v-if="sortedRows.length === 0">
          <td :colspan="columns.length" class="px-6 py-4 text-center text-gray-500">
            No results
          </td>
        </tr>

        <!-- DATA ROWS: only the rows for the current page (pagedRows), not all of them. -->
        <!-- Hovering highlights the row using the hoverRow class from index.css. -->
        <!-- Clicking or pressing Enter sends the row up to the parent as a row-click event. -->
        <tr
          v-for="(row, index) in pagedRows"
          :key="row._id || row.id || index"
          :class="{ hoverRow: hoverIndex === index }"
          class="cursor-pointer"
          tabindex="0"
          @mouseenter="hoverIndex = index"
          @mouseleave="hoverIndex = null"
          @click="$emit('row-click', row)"
          @keydown.enter="$emit('row-click', row)"
        >
          <!-- One cell per column. cellValue decides what text to show. -->
          <td
            v-for="col in columns"
            :key="col.field"
            class="px-6 py-4 whitespace-nowrap"
          >
            {{ cellValue(row, col) }}
          </td>
        </tr>
      </tbody>
    </table>

    <!-- PAGINATION BAR: hidden when there are no rows. -->
    <div v-if="sortedRows.length > 0" class="flex items-center justify-between mt-4">
      <!-- The "Showing 1-10 of 15" text. startIndex is zero-based, so we add 1. -->
      <span class="text-sm text-gray-700">
        Showing {{ startIndex + 1 }}-{{ endIndex }} of {{ sortedRows.length }}
      </span>
      <div class="flex gap-2">
        <!-- Buttons turn off on the first and last page so you can't go out of range. -->
        <button :disabled="page === 1" @click="page--">Previous</button>
        <span class="px-2">Page {{ page }} of {{ totalPages }}</span>
        <button :disabled="page === totalPages" @click="page++">Next</button>
      </div>
    </div>
  </div>
</template>

<script setup>
// ref = a value that Vue watches. computed = a value calculated from other values.
// watch = run some code when a value changes.
import { ref, computed, watch } from 'vue'

// PROPS: the data the parent hands to this component.
// columns: list of { label, field, render? } describing each column.
// rows: the actual data to show.
// pageSize: rows per page, defaults to 10.
const props = defineProps({
  columns: { type: Array, required: true },
  rows: { type: Array, default: () => [] },
  pageSize: { type: Number, default: 10 }
})

// EVENTS: this component can tell its parent "a row was clicked."
defineEmits(['row-click'])

// STATE: things that change while the user interacts.
// sortField is which column is sorted. sortDir is 'asc', 'desc', or null (unsorted).
const sortField = ref(null)
const sortDir = ref(null)
const page = ref(1)
const hoverIndex = ref(null)

// SORT CLICK LOGIC: each click on a header moves through
// ascending -> descending -> unsorted. Clicking a new column starts at ascending.
// We also jump back to page 1 so you don't land on an empty page.
function toggleSort(field) {
  if (sortField.value !== field) {
    sortField.value = field
    sortDir.value = 'asc'
  } else if (sortDir.value === 'asc') {
    sortDir.value = 'desc'
  } else {
    sortField.value = null
    sortDir.value = null
  }
  page.value = 1
}

// CELL TEXT: if the column has a custom render function, use it.
// Otherwise just show the value stored under that field name.
function cellValue(row, col) {
  return col.render ? col.render(row) : row[col.field]
}

// SORTED ROWS: if nothing is sorted, use the rows as they came in.
// Otherwise make a copy ([...props.rows]) and sort the copy, so we never
// change the parent's original data. dir is 1 for ascending, -1 for descending.
const sortedRows = computed(() => {
  if (!sortField.value) return props.rows
  const field = sortField.value
  const dir = sortDir.value === 'asc' ? 1 : -1
  return [...props.rows].sort((a, b) => {
    const av = a[field] ?? ''
    const bv = b[field] ?? ''
    if (av < bv) return -1 * dir
    if (av > bv) return 1 * dir
    return 0
  })
})

// PAGING MATH: how many pages exist, and which slice of rows to show.
// Math.max(1, ...) makes sure there is always at least 1 page.
const totalPages = computed(() =>
  Math.max(1, Math.ceil(sortedRows.value.length / props.pageSize))
)
const startIndex = computed(() => (page.value - 1) * props.pageSize)
const endIndex = computed(() =>
  Math.min(startIndex.value + props.pageSize, sortedRows.value.length)
)

// PAGED ROWS: the small slice of sorted rows that actually appears on screen.
const pagedRows = computed(() =>
  sortedRows.value.slice(startIndex.value, endIndex.value)
)

// STALE ROWS FIX: whenever the parent gives us new rows (like after a search),
// go back to page 1 so old results never linger on screen.
watch(
  () => props.rows,
  () => {
    page.value = 1
  }
)
</script>