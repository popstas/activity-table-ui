<template>
  <div class="calendar-month">
    <Stats v-if="itemsYear && indicators" :items="itemsMonth" :summaryLabel="`Всего ${currentPeriod}`"></Stats>
    <v-calendar
      @update:from-page="updateFrom"
      @update:to-page="updateTo"
      is-expanded
      :rows="months"
      :step="1"
      :attributes="attributes"
    ></v-calendar>
  </div>
</template>

<script>
export default {
  components: {
  },

  props: ['items', 'indicators', 'months'],

  data() {
    const d = new Date();
    return {
      itemsYear: [],
      currentYear: new Date().getFullYear(),
      from: new Date(d.getFullYear(), d.getMonth()),
      to: new Date(),
      attributes: [],
    }
  },

  watch: {
    indicators(val) {
      // this.itemsYear = this.getYearItems(this.currentYear);
    }
  },

  computed: {
    itemsMonth() {
      let items

      // indicators
      if(!this.indicators) {
        items = this.items
      }
      else {
        items = this.items.filter((item) => {
          return this.indicators.includes(item.name)
        })
      }

      // const from = new Date(this.from.getFullYear(), this.from.getMonth() - this.lastMonths);
      items = items.filter((item) => {
        return item.startDate >= this.from && item.endDate < this.to
      })

      // replace color
      items = items.map((item) => {
        item.color = item.color2;
        return item;
      })

      return items
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

    currentPeriod() {
      const format = (d) => {
        const day = `${d.getDate()}`.padStart(2, '0')
        const month = `${d.getMonth()+1}`.padStart(2, '0')
        return `${day}.${month}.${d.getFullYear()}`
      }
      if (!this.from || !this.to) return '';
      const from = format(this.from);
      const to = format(new Date(this.to.getTime() - 1));
      return `${from} - ${to}`
    }
  },

  methods: {
    updateFrom(val) {
      this.from = new Date(val.year, val.month - 1);
      this.attributes = this.getAttributes()
      console.log('updateFrom: ', this.from);
      console.log('updateFrom: ', val);
    },

    updateTo(val) {
      this.to = new Date(val.year, val.month - 0);
      this.attributes = this.getAttributes()
      console.log('updateTo: ', this.to);
      console.log('updateTo: ', val);
    },

    getAttributes() {
      const attributes = this.itemsMonth.map(item => {
        return {
          dates: item.startDate,
          // content: 'red',
          dot: item.color2,
          customData: {
            title: item.indicator
          },
          /* highlight: {
            color: item.color2 || 'green',
            fillMode: 'light',
          }, */
        }
      })
      console.log('attributes: ', attributes)
      return attributes
    },
  },

  mounted() {
    // this.attributes = this.getAttributes()
  },
}
</script>
<style>

.calendar-month {
  
}
</style>
