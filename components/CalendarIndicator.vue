<template>
  <div class="calendar-indicator">
    <Stats v-if="itemsYear && indicators" :items="itemsYear" :summaryLabel="`Всего (${currentYear})`"></Stats>
    <Calendar 
      language="ru"
      :dataSource="dataSource"
      :min-date="minDate"
      :max-date="maxDate"
      :custom-data-source-renderer="renderDay"
      render-style="custom"
      @year-changed="yearChanged"
      :year="new Date().getFullYear() - 1"
    ></Calendar>
    <Calendar 
      language="ru"
      :dataSource="dataSource"
      :min-date="minDate"
      :max-date="maxDate"
      :custom-data-source-renderer="renderDay"
      render-style="custom"
      @year-changed="yearChanged"
    ></Calendar>

  </div>
</template>

<script>
import Calendar from 'v-year-calendar'
import 'v-year-calendar/locales/v-year-calendar.ru'

export default {
  components: {
    Calendar,
  },

  props: ['items', 'indicators'],

  data() {
    return {
      itemsYear: [],
      currentYear: new Date().getFullYear(),
    }
  },

  watch: {
    indicators(val) {
      this.itemsYear = this.getYearItems(this.currentYear);
    }
  },

  computed: {
    dataSource() {
      if(!this.indicators) return this.items
      return this.items.filter((item) => {
        return this.indicators.includes(item.name)
      })
    },

    minDate() {
      let minDate = Date.now()
      for(let item of this.dataSource) {
        if (item.startDate < minDate) minDate = item.startDate
      }
      return new Date(minDate)
    },

    maxDate() {
      let maxDate = Date.now()
      for(let item of this.dataSource) {
        if (item.endDate > maxDate) maxDate = item.endDate
      }
      return new Date(maxDate)
    },
  },

  methods: {
    renderDay(element, currentDate, events) {
      // console.log('element: ', element);
      // console.log('currentDate: ', currentDate);
      // console.log('events: ', events);

      const uniqueColors = [];
      for (let event of events) {
        if (event.color && !uniqueColors.includes(event.color)) {
          uniqueColors.push(event.color)
        }
      }

      // const maxSizePercent = 50
      const maxSizePercent = Array.isArray(this.indicators) ? 100 / this.indicators.length : 50
      const sizePercent = uniqueColors.length * maxSizePercent > 100 ? 100 / uniqueColors.length : maxSizePercent

      const colors = []
      /* for (let ind in uniqueColors) {
        const color = uniqueColors[ind]
        const from = ind * sizePercent
        const to = from + sizePercent

        colors.push(`${color} ${from}%, ${color} ${to}%`)
      } */

      const indicatorsCount = Object.keys(this.indicators).length
      const isSummary = indicatorsCount > 2
      let summaryValue = 0

      for (let ind in this.indicators) {
        const indicator = this.indicators[ind]
        const foundEvent = events.find(ev => ev.indicator == indicator)
        if (foundEvent) summaryValue++;
        const color = foundEvent ? foundEvent.color : '#fff'
        const from = ind * sizePercent
        const to = from + sizePercent
        if (!isSummary) {
          colors.push(`${color} ${from}%, ${color} ${to}%`)
        }
      }

      // одна заливка, когда много мелких
      if (isSummary) {
        const from = 0
        const to = summaryValue / indicatorsCount * 100
        const h = Math.round(120 * to / 100)
        const color = `hsl(${h}, 80%, 50%)`
        colors.push(`${color} ${from}%, ${color} ${to}%`)
      }

      // last
      const fromLast = uniqueColors.length * sizePercent
      colors.push(`#fff ${fromLast}%`)

      element.style.background = `linear-gradient(180deg, ${colors.join(', ')})`
      // element
    },

    yearChanged(e) {
      this.currentYear = e.currentYear
      this.itemsYear = this.getYearItems(this.currentYear)
    },

    getYearItems(year) {
      const yearFromDate = new Date(year, 0, 1)
      const yearToDate = new Date(year + 1, 0, 1)
      // console.log('yearFromDate: ', yearFromDate)
      // console.log('yearToDate: ', yearToDate)

      return this.dataSource.filter((item) => {
        return item.startDate > yearFromDate && item.endDate < yearToDate
      });
    },
  },
}
</script>
<style>
/* .calendar table td, .calendar table th {
  width: 32px;
  height: 32px;
} */
.calendar {
  padding: 0;
}
.calendar .months-container .month-container {
  height: 220px;
}
.calendar-indicator {
  text-align: center;
}
.calendar table.month td.day .day-content {
  /* margin: 2px; */
  border-radius: 0;
}
</style>
