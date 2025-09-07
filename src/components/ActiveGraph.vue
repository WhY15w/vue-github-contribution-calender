<template>
  <div
    ref="scrollContainerChart"
    class="contribution-chart w-full overflow-auto p-5 pb-2"
  >
    <div class="content min-w-3xl">
      <table class="w-full border-spacing-1">
        <!-- 月份表头 -->
        <thead>
          <tr>
            <th
              class="w-10"
              aria-hidden="true"
            ></th>
            <template v-for="month in months">
              <th
                :colspan="month.colspan"
                class="relative pb-1 text-left text-xs font-normal text-gray-700"
              >
                {{ monthMap[month.month] }}
              </th>
            </template>
          </tr>
        </thead>

        <tbody>
          <template
            v-for="(row, rowIndex) in rowData"
            :key="rowIndex"
          >
            <tr>
              <td class="w-10 text-xs text-gray-700">
                {{ rowIndex % 2 ? weekMap[rowIndex] : '' }}
              </td>

              <template
                v-for="item in row"
                :key="item.date || item.id"
              >
                <td
                  v-if="!item.ignore"
                  :data-date="item.date"
                  :title="`${item.date}, ${item.count}次提交`"
                  class="h-4 w-4 text-xs"
                >
                  <div
                    class="h-3 w-3 rounded-xs border transition-all duration-200 hover:scale-110 hover:cursor-pointer"
                    :style="{ backgroundColor: item.backgroundColor }"
                    @click="emit('click', item)"
                  ></div>
                </td>
                <td
                  v-else
                  class="h-3 w-3"
                  style="background-color: transparent"
                ></td>
              </template>
            </tr>
          </template>
        </tbody>
      </table>
    </div>
  </div>
  <div
    class="tfoot mt-2 mb-4 flex w-full items-center justify-between text-xs text-gray-500"
  >
    <div class="description pl-7">总计提交: {{ total }}</div>
    <div class="colors flex items-center pr-7">
      <span class="mr-2">Less</span>
      <template v-for="color in levelColorMap">
        <span
          class="ml-1 inline-block h-3 w-3 rounded-xs border"
          :style="{ backgroundColor: color }"
        ></span>
      </template>
      <span class="ml-2">More</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
  contributions: {
    type: Array,
    required: true,
    default: () => [],
  },
  total: {
    type: Number,
    required: true,
    default: 0,
  },
  colorLevels: {
    type: Object,
    default: () => ({
      0: '#eff2f5',
      1: '#aceebb',
      2: '#4ac26b',
      3: '#2da44e',
      4: '#116329',
    }),
  },
  lang: {
    type: String,
    default: 'zh', // 'en' or 'zh'
  },
})

const emit = defineEmits(['click'])

const monthTranslations = {
  zh: {
    1: '1月',
    2: '2月',
    3: '3月',
    4: '4月',
    5: '5月',
    6: '6月',
    7: '7月',
    8: '8月',
    9: '9月',
    10: '10月',
    11: '11月',
    12: '12月',
  },
  en: {
    1: 'Jan',
    2: 'Feb',
    3: 'Mar',
    4: 'Apr',
    5: 'May',
    6: 'Jun',
    7: 'Jul',
    8: 'Aug',
    9: 'Sep',
    10: 'Oct',
    11: 'Nov',
    12: 'Dec',
  },
}

const weekTranslations = {
  zh: {
    0: '周日',
    1: '周一',
    2: '周二',
    3: '周三',
    4: '周四',
    5: '周五',
    6: '周六',
  },
  en: { 0: 'Sun', 1: 'Mon', 2: 'Tue', 3: 'Wed', 4: 'Thu', 5: 'Fri', 6: 'Sat' },
}

const monthMap = monthTranslations[props.lang === 'zh' ? 'zh' : 'en']
const weekMap = weekTranslations[props.lang === 'zh' ? 'zh' : 'en']

const months = ref([])
const rowData = ref([])
const levelColorMap = ref(props.colorLevels)

const scrollContainerChart = ref(null)

// 获取背景颜色
const getBackgroundColor = (level) => {
  return levelColorMap.value[level] || '#ebedf0'
}

// 处理行数据
const processRowData = (data) => {
  const rowResultData = []

  for (let i = 0; i < 7; i++) {
    const rowDataTmp = []

    for (let j = 0; j < data.length; j++) {
      if (data[j].length > i) {
        rowDataTmp.push({
          ...data[j][i],
          ignore: false,
          backgroundColor: getBackgroundColor(
            Number(data[j][i].intensity) || 0,
          ),
        })
      } else {
        rowDataTmp.push({
          id: Math.random(),
          date: '',
          count: 0,
          intensity: 0,
          ignore: true,
          backgroundColor: getBackgroundColor(0),
        })
      }
    }

    rowResultData.push(rowDataTmp)
  }

  rowData.value = rowResultData
}

// 处理月份数据
const processMonthData = (data) => {
  const monthsData = []
  let currentMonth = null

  for (const column of data) {
    if (column.length === 0) continue
    const [year, month] = column[0].date.split('-').map(Number)

    if (
      currentMonth &&
      currentMonth.year === year &&
      currentMonth.month === month
    ) {
      currentMonth.colspan += 1
    } else {
      currentMonth = { colspan: 1, year, month }
      monthsData.push(currentMonth)
    }
  }

  months.value = monthsData
}

// 处理数据
const processData = () => {
  processMonthData(props.contributions)
  processRowData(props.contributions)
}

onMounted(() => {
  processData()
})

watch(
  () => props.contributions,
  () => {
    processData()
    if (scrollContainerChart.value) {
      // 滚动到最右侧
      scrollContainerChart.value.scrollLeft =
        scrollContainerChart.value.scrollWidth
    }
  },
)
</script>

<style scoped>
@media (max-width: 550px) {
  .contribution-chart thead {
    display: table-header-group;
  }

  .contribution-chart .text-xs {
    font-size: 9px;
  }
}
</style>
