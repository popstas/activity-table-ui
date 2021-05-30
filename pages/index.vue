<template>
  <div class="container">
    <div class="indicator-select">
      <el-checkbox :indeterminate="isIndeterminate" v-model="checkAll" @change="handleCheckAllChange">Все</el-checkbox>
      <el-checkbox-group v-model="currentIndicators" @change="handleCheckedIndicatorsChange">
        <el-checkbox
          v-for="indicator in indicators"
          :key="indicator"
          :label="indicator"
          :value="indicator"
        ></el-checkbox>
      </el-checkbox-group>
    </div>
    <div>
      <CalendarIndicator v-if="currentIndicators && currentIndicators.length > 0" :items="items" :indicators="currentIndicators"></CalendarIndicator>
    </div>


    <!-- <div>
      <h3>Все</h3>
      <CalendarIndicator :items="items"></CalendarIndicator>
    </div> -->

    <div class="calendar-block">
      <h2>Здоровье</h2>
      <CalendarIndicator :items="items" :indicators="categories['Здоровье']"></CalendarIndicator>
    </div>

    <div class="calendar-block">
      <h2>Продуктивность</h2>
      <CalendarIndicator :items="items" :indicators="categories['Продуктивность']"></CalendarIndicator>
    </div>

    <h2>Динамика изменения</h2>
    <el-row class="stats-table">
      <el-col :span="12">
        <h3>7 дней</h3>
        <div v-for="(indicators, name) in categories" :key="name">
          <h4>{{ name }}</h4>
          <Stats :totalDays="7" :items="items7days.filter(i => indicators.includes(i.indicator))"></Stats>
        </div>
      </el-col>
      <el-col :span="3">
        <h3>14 дней</h3>
        <div v-for="(indicators, name) in categories" :key="name">
          <h4>&nbsp;</h4>
          <Stats :noLabels="true" :totalDays="14" :items="items14days.filter(i => indicators.includes(i.indicator))"></Stats>
        </div>
      </el-col>
      <el-col :span="3">
        <h3>30 дней</h3>
        <div v-for="(indicators, name) in categories" :key="name">
          <h4>&nbsp;</h4>
          <Stats :noLabels="true" :totalDays="30" :items="items30days.filter(i => indicators.includes(i.indicator))"></Stats>
        </div>
      </el-col>
      <el-col :span="3">
        <h3>60 дней</h3>
        <div v-for="(indicators, name) in categories" :key="name">
          <h4>&nbsp;</h4>
          <Stats :noLabels="true" :totalDays="60" :items="getItemsByIndicatorsAndDays(indicators, 60)"></Stats>
        </div>
      </el-col>
      <el-col :span="3">
        <h3>90 дней</h3>
        <div v-for="(indicators, name) in categories" :key="name">
          <h4>&nbsp;</h4>
          <Stats :noLabels="true" :totalDays="90" :items="getItemsByIndicatorsAndDays(indicators, 90)"></Stats>
        </div>
      </el-col>
    </el-row>

    <!-- <div v-for="indicator of indicators" :key="indicator" class="calendar-indicator">
      <h3>{{ indicator }}</h3>
      <CalendarIndicator :items="items" :indicators="[indicator]"></CalendarIndicator>
    </div> -->
  </div>
</template>

<script>
import CalendarIndicator from '@/components/CalendarIndicator'
import { createHash } from 'crypto'

let metrics = require('../../activity-table/data/items.json')
metrics = metrics.filter(m => !!m.value)

// const colors = ['#2C8FC9', '#9CB703', '#F5BB00', '#FF4A32', '#B56CE2', '#45A597'];
const colors = [
  '#4472C4',
  '#ED7D31',
  '#A5A5A5',
  '#FFC000',
  '#5B9BD5',
  '#70AD47',
  '#96BCFF',
  '#9E480E',
  '#997300',
  '#F1975A',
  '#9DC3E6',
  '#E5C300',
]

// angular
colors.push(...['#9CB703', '#F5BB00', '#FF4A32', '#B56CE2', '#45A597']);

// бледные
// colors.push(...['#ffadad', '#ffd6a5', '#fdffb6', '#caffbf', '#9bf6ff', '#a0c4ff', '#bdb2ff', '#ffc6ff']);


const excludedIndicators = [
  'Welltory батарейка',
  'Welltory уровень стресса',
  'Разблокировок смартфона',
  'Доволен результатами дня',
  'Vk + telegram за прошлую неделю, часов',
  'Читал новости за неделю',
  'Запланировал неделю',
  'Активные тренировки',
  'Средняя оценка за неделю',
  'Время на итоги недели',
]

export default {
  components: {
    CalendarIndicator
  },

  data() {
    return {
      currentIndicators: [],
      checkAll: false,
      isIndeterminate: false,
      categories: {
        'Здоровье': [
          'Проснулся свежим и бодрым',
          'Проснулся вовремя',
          'Зарядка',
          'Медитация',
          'Лёг спать до полуночи',
          'Заснул быстрее 30 минут',
          'Не курил',
        ],
        'Продуктивность': [
          'Проснулся свежим и бодрым',
          'Встал с первым будильником',
          'Проснулся вовремя',
          'Сделал задачу до проверки входящих',
          'Сделал негорящую задачу',
          'Делал задачи из основного рабочего списка',
          'Делал задачи из домашнего списка',
          'Не читал свежие новости (включая vk)',
          'Не чатился дольше 10 минут',
        ],
        'Удовольствия': [
          'Записал сон',
          'Бег/велик/лыжи',
        ],
      }
    }
  },

  computed: {
    indicatorColors() {
      const indicatorColors = {};
      for(let indicator of this.indicators) {
        const hash = createHash('md5').update(indicator).digest('hex')
        const hashInt = Number(BigInt(`0x${hash.substring(0, 16)}`)) / 1000000
        const colorInd = Math.floor(hashInt % colors.length);
        // console.log('hashInt: ', hashInt);
        // console.log('colorInd: ', colorInd);

        indicatorColors[indicator] = colors[colorInd]
      }
      // console.log('indicatorColors: ', indicatorColors);
      return indicatorColors;
    },

    items() {
      return metrics.
      filter(m => !excludedIndicators.includes(m.indicator)).
      map((item, id) => {
        const startDate = new Date(`${item.date} 00:00:00`)
        const endTime = startDate.getTime() + 1 * 3600 * 1000 // +1 hour

        // const color = '#9cb703';

        const color = this.indicatorColors[item.indicator];

        return {
          id,
          name: item.indicator,
          indicator: item.indicator,
          startDate,
          endDate: new Date(endTime),
          color
        }
      })
    },

    items7days() {
      return this.getItemsForLastDays(this.items, 7);
    },
    items14days() {
      return this.getItemsForLastDays(this.items, 14);
    },
    items30days() {
      return this.getItemsForLastDays(this.items, 30);
    },

    indicators() {
      const indicators = []

      for (let m of metrics) {
        if (!indicators.includes(m.indicator) && !excludedIndicators.includes(m.indicator)) {
          indicators.push(m.indicator)
        }
      }
      // console.log('indicators: ', indicators);
      return indicators
    },
  },

  methods: {
    handleCheckAllChange(val) {
      this.currentIndicators = val ? this.indicators : [];
      this.isIndeterminate = false;
    },

    handleCheckedIndicatorsChange(value) {
      let checkedCount = value.length;
      this.checkAll = checkedCount === this.currentIndicators.length;
      this.isIndeterminate = checkedCount > 0 && checkedCount < this.indicators.length;
    },

    getItemsForLastDays(items, days = 30) {
      const fromDate = new Date(Date.now() - days * 86400 * 1000);
      const toDate = new Date();
      // console.log('fromDate: ', fromDate);
      // console.log('toDate: ', toDate);
      return items.filter((item) => {
        return item.startDate > fromDate && item.endDate < toDate
      });
    },

    getItemsByIndicatorsAndDays(indicators, days) {
      return this.
        getItemsForLastDays(this.items, days).
        filter(i => indicators.includes(i.indicator))
    },
  },
}
</script>

<style>
.container {
  margin: 0 auto;
  padding: 15px 0 0;
  min-height: 100vh;
}

.day-start {
  /* background-color: #9cb703 !important; */
}

.el-checkbox-group {
  text-align: left;
  columns: 2;
  margin-bottom: 15px;
}
@media (max-width: 640px) {
  .el-checkbox-group {
    columns: 1;
  }
  .stats-table .stats-item-color,
  .stats-table .stats-item-counters {
    display: none;
  }
}
.el-checkbox {
  display: block;
}
.el-checkbox__label {
  font-size: 20px;
  line-height: 1.5em;
}

.calendar-block {
  /* text-align: center; */
  margin-top: 120px;
}

.indicator-select {
  text-align: left;
}

h2 {
  font-size: 30px;
  margin: 120px 0 15px;
}
h3 {
  font-size: 24px;
  line-height: 2em;
}
</style>
