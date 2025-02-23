<script setup lang="ts">
import debounce from 'lodash.debounce'
import { computed, onMounted, ref, watch } from 'vue'
import { initDropdowns } from 'flowbite'
import {
  kList, kListItem,
} from 'konsta/vue'
import { useI18n } from 'vue-i18n'
import type { MobileColType, TableColumn } from './comp_def'
import IconNext from '~icons/ic/round-keyboard-arrow-right'
import IconSort from '~icons/lucide/chevrons-up-down'
import IconSortUp from '~icons/lucide/chevron-up'
import IconSortDown from '~icons/lucide/chevron-down'
import IconSearch from '~icons/ic/round-search'
import IconReload from '~icons/tabler/reload'
import IconDown from '~icons/ic/round-keyboard-arrow-down'
import IconFilter from '~icons/system-uicons/filtering'
import IconFastForward from '~icons/ic/round-keyboard-double-arrow-right'
import IconPrev from '~icons/ic/round-keyboard-arrow-left'
import IconFastBackward from '~icons/ic/round-keyboard-double-arrow-left'

interface Props {
  rowClick?: boolean
  isLoading?: boolean
  filterText?: string
  filters?: { [key: string]: boolean }
  searchPlaceholder?: string
  search?: string
  total: number
  currentPage: number
  columns: TableColumn[]
  elementList: { [key: string]: any }[]
}

const props = defineProps<Props>()
const emit = defineEmits([
  'reload', 'reset', 'next', 'prev', 'fastForward', 'fastBackward',
  'update:search', 'update:filters', 'update:columns', 'update:currentPage',
  'filterClick', 'rowClick', 'sortClick'])
const { t } = useI18n()
const searchVal = ref(props.search || '')
// const sorts = ref<TableSort>({})
// get columns from elementList

const offset = computed(() => {
  if (!props.elementList)
    return 0
  return props.elementList.length
})

const filterList = computed(() => {
  if (!props.filters)
    return []
  return Object.keys(props.filters)
})
const filterActivated = computed(() => {
  if (!props.filters)
    return []
  return Object.keys(props.filters).reduce((acc, key) => {
    if (props.filters![key])
      acc += 1
    return acc
  }, 0)
})

function sortClick(key: number) {
  if (!props.columns[key].sortable)
    return
  let sortable = props.columns[key].sortable
  if (sortable === 'asc')
    sortable = 'desc'
  else if (sortable === 'desc')
    sortable = true
  else
    sortable = 'asc'
  const newColumns = [...props.columns]
  newColumns[key].sortable = sortable
  emit('update:columns', newColumns)
}

watch(props.columns, () => {
  emit('reload')
})
if (props.filters) {
  watch(props.filters, () => {
    emit('reload')
  })
}

watch(searchVal, debounce(() => {
  emit('update:search', searchVal.value)
  emit('reload')
}, 500))

function displayValueKey(elem: any, col: TableColumn | undefined) {
  if (!col)
    return ''
  return col.displayFunction ? col.displayFunction(elem) : elem[col.key]
}
const displayElemRange = computed(() => {
  const begin = (props.currentPage - 1) * props.elementList.length
  const end = begin + props.elementList.length
  return `${begin}-${end}`
})

function findMobileCol(name: MobileColType) {
  return props.columns ? props.columns.find(col => col.mobile === name) : undefined
}

async function prev() {
  console.log('prev')
  if (props.currentPage > 1) {
    emit('prev')
    emit('update:currentPage', props.currentPage - 1)
    emit('reload')
  }
}
async function next() {
  console.log('next')
  if (props.currentPage < Math.ceil(props.total / offset.value)) {
    emit('next')
    emit('update:currentPage', props.currentPage + 1)
    emit('reload')
  }
}
async function fastForward() {
  console.log('fastForward')
  if (props.currentPage < Math.ceil(props.total / offset.value)) {
    emit('fastForward')
    emit('update:currentPage', Math.ceil(props.total / offset.value))
    emit('reload')
  }
}
async function fastBackward() {
  console.log('fastBackward')
  if (props.currentPage > 1) {
    emit('fastBackward')
    emit('update:currentPage', 1)
    emit('reload')
  }
}

onMounted(() => {
  initDropdowns()
})
</script>

<template>
  <div class="relative pb-4 pb-20 overflow-x-auto md:pb-0">
    <div class="flex items-start justify-between pb-4 md:items-center">
      <div class="flex mb-2 md:mb-0">
        <button class="relative mr-2 inline-flex items-center border border-gray-300 rounded-lg bg-white px-3 py-1.5 text-sm font-medium text-gray-500 dark:border-gray-600 dark:bg-gray-800 hover:bg-gray-100 dark:text-white focus:outline-none focus:ring-4 focus:ring-gray-200 dark:hover:border-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-700" type="button" @click="emit('reset')">
          <IconReload v-if="!isLoading" class="m-1 mr-2" />
          <Spinner v-else size="w-[16.8px] h-[16.8px] m-1 mr-2" />
          <span class="hidden text-sm md:block">{{ t('reload') }}</span>
        </button>
        <button v-if="filterText && filterList.length" id="dropdownRadioButton" data-dropdown-offset-skidding="100" data-dropdown-toggle="dropdownRadio" class="relative inline-flex items-center border border-gray-300 rounded-lg bg-white px-3 py-1.5 text-sm font-medium text-gray-500 dark:border-gray-600 dark:bg-gray-800 hover:bg-gray-100 dark:text-white focus:outline-none focus:ring-4 focus:ring-gray-200 dark:hover:border-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-700" type="button">
          <div v-if="filterActivated" class="absolute inline-flex items-center justify-center w-6 h-6 text-xs font-bold text-white bg-red-500 border-2 border-white rounded-full -right-2 -top-2 dark:border-gray-900">
            <!-- uppercase first letter in tailwind -->
            {{ filterActivated }}
          </div>
          <IconFilter class="m-1 mr-2" />
          <span class="hidden md:block">{{ t(filterText) }}</span>
          <IconDown class="hidden m-1 ml-2 md:block" />
        </button>
        <!-- Dropdown menu -->
        <div v-if="filterText && filterList.length" id="dropdownRadio" class="z-50 hidden w-48 bg-white border divide-y divide-gray-100 rounded-lg shadow dark:bg-gray-700 dark:divide-gray-600" data-popper-reference-hidden="" data-popper-escaped="" data-popper-placement="top">
          <div class="block px-4 py-3 text-sm text-gray-900 md:hidden dark:text-white">
            <div>{{ t(filterText) }}</div>
          </div>
          <ul class="p-3 space-y-1 text-sm text-gray-700 dark:text-gray-200" aria-labelledby="dropdownRadioButton">
            <li v-for="(f, i) in filterList" :key="i">
              <div class="flex items-center p-2 rounded hover:bg-gray-100 dark:hover:bg-gray-600">
                <input :id="`filter-radio-example-${i}`" v-model="(filters as any)[f]" type="checkbox" :name="`filter-radio-${i}`" class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 dark:border-gray-600 dark:bg-gray-700 focus:ring-2 focus:ring-blue-500 dark:ring-offset-gray-800 dark:focus:ring-blue-600 dark:focus:ring-offset-gray-800" @click="emit('filterClick', { clicked: f, filters })">
                <label :for="`filter-radio-example-${i}`" class="w-full ml-2 text-sm font-medium text-gray-900 rounded dark:text-gray-300 first-letter:uppercase">{{ t(f) }}</label>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <!-- </div> -->
      <div class="relative w-70 md:w-auto">
        <label for="table-search" class="sr-only">{{ searchPlaceholder }}</label>
        <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
          <IconSearch class="w-5 h-5 text-gray-500 dark:text-gray-400" />
        </div>
        <input id="table-search" v-model="searchVal" type="text" class="block w-full p-2 pl-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 md:w-80 dark:border-gray-600 focus:border-blue-500 dark:bg-gray-700 dark:text-white focus:ring-blue-500 dark:focus:border-blue-500 dark:focus:ring-blue-500 dark:placeholder-gray-400" :placeholder="searchPlaceholder">
      </div>
    </div>
    <div class="hidden md:block">
      <table class="w-full text-sm text-left text-gray-500 dark:text-gray-400">
        <thead class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
          <tr>
            <th v-for="(col, i) in columns" :key="i" scope="col" class="px-6 py-3" :class="{ 'cursor-pointer': col.sortable }" @click="sortClick(i)">
              <div class="flex items-center">
                {{ col.label }}
                <div v-if="col.sortable">
                  <IconSortUp v-if="col.sortable === 'asc'" />
                  <IconSortDown v-else-if="col.sortable === 'desc'" />
                  <IconSort v-else />
                </div>
              </div>
            </th>
          </tr>
        </thead>
        <tbody v-if="!isLoading">
          <tr
            v-for="(elem, i) in elementList" :key="i"
            :class="{ 'cursor-pointer': rowClick }"
            class="bg-white border-b dark:border-gray-700 dark:bg-gray-800 hover:bg-gray-50 dark:hover:bg-gray-600"
            @click="emit('rowClick', elem)"
          >
            <template v-for="(col, _y) in columns" :key="`${i}_${_y}`">
              <th v-if="col.head" scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                {{ displayValueKey(elem, col) }}
              </th>
              <td v-else-if="col.icon" :class="col.class" class="px-6 py-4 cursor-pointer" @click.stop="col.onClick ? col.onClick(elem) : () => {}" v-html="col.icon" />
              <td v-else class="px-6 py-4">
                {{ displayValueKey(elem, col) }}
              </td>
            </template>
          </tr>
        </tbody>
        <tbody v-else>
          <tr v-for="i in 10" :key="i" class="max-w-sm animate-pulse">
            <td v-for="(col, y) in columns" :key="`${i}_${y}`" class="px-6 py-4">
              <div class="max-w-[300px] rounded-full bg-gray-200 dark:bg-gray-700" :class="{ 'mb-4 h-2.5': col.head, 'h-2 mb-2.5': !col.head }" />
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <kList class="block !my-0 md:hidden">
      <kListItem
        v-for="(elem, i) in elementList" :key="i"
        :title="displayValueKey(elem, findMobileCol('title'))"
        :footer="displayValueKey(elem, findMobileCol('footer'))"
        :header="displayValueKey(elem, findMobileCol('header'))"
        @click="emit('rowClick', elem)"
      >
        <template #after>
          <div v-if="findMobileCol('after')?.icon" @click.stop="findMobileCol('after')?.onClick" v-html="findMobileCol('after')?.icon" />
          <span v-else>{{ displayValueKey(elem, findMobileCol('after')) }}</span>
        </template>
      </kListItem>
    </kList>
    <nav class="fixed bottom-0 left-0 z-40 flex items-center justify-between w-full p-4 bg-white md:relative dark:bg-gray-900 md:bg-transparent md:pt-4 dark:md:bg-transparent" aria-label="Table navigation">
      <span class="text-sm font-normal text-gray-500 dark:text-gray-400"><span class="hidden md:inline-block">Showing</span> <span class="font-semibold text-gray-900 dark:text-white">{{ displayElemRange }}</span> of <span class="font-semibold text-gray-900 dark:text-white">{{ total }}</span></span>
      <ul class="inline-flex items-center -space-x-px">
        <li>
          <button class="block px-3 py-2 ml-0 leading-tight text-gray-500 bg-white border border-gray-300 rounded-l-lg dark:border-gray-700 dark:bg-gray-800 hover:bg-gray-100 dark:text-gray-400 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white" @click="fastBackward">
            <span class="sr-only">Fast Backward</span>
            <IconFastBackward />
          </button>
        </li>
        <li>
          <button class="block px-3 py-2 ml-0 leading-tight text-gray-500 bg-white border border-gray-300 dark:border-gray-700 dark:bg-gray-800 hover:bg-gray-100 dark:text-gray-400 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white" @click="prev">
            <span class="sr-only">Previous</span>
            <IconPrev />
          </button>
        </li>
        <li>
          <button aria-current="page" class="z-10 px-3 py-2 leading-tight text-blue-600 border border-blue-300 bg-blue-50 dark:border-gray-700 dark:bg-gray-700 hover:bg-blue-100 dark:text-white hover:text-blue-700">
            {{ currentPage }}
          </button>
        </li>
        <li>
          <button class="block px-3 py-2 leading-tight text-gray-500 bg-white border border-gray-300 dark:border-gray-700 dark:bg-gray-800 hover:bg-gray-100 dark:text-gray-400 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white" @click="next">
            <span class="sr-only">Next</span>
            <IconNext />
          </button>
        </li>
        <li>
          <button class="block px-3 py-2 leading-tight text-gray-500 bg-white border border-gray-300 rounded-r-lg dark:border-gray-700 dark:bg-gray-800 hover:bg-gray-100 dark:text-gray-400 hover:text-gray-700 dark:hover:bg-gray-700 dark:hover:text-white" @click="fastForward">
            <span class="sr-only"> Fast Forward </span>
            <IconFastForward />
          </button>
        </li>
      </ul>
    </nav>
  </div>
</template>
