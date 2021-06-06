<template>
  <div class="stats">
    <div class="stats-total">
      <StatsItem 
        :indicator="noLabels ? '' : (summaryLabel || 'Всего')"
        :indicatorStats="{total: stats.totalValue }"
        :totalDays="stats.totalDays * Object.keys(stats.indicators).length"
        :isSummary="true"
        :color="noLabels ? '' : '#999'"
      >
      </StatsItem>
    </div>

    <StatsItem v-for="(indicatorStats, indicator) in stats.indicators" :key="indicator"
    :indicator="noLabels ? '' : indicator"
    :indicatorStats="indicatorStats"
    :totalDays="stats.totalDays"
    :color="noLabels ? '' : indicatorStats.color"
    >
    </StatsItem>
  </div>
</template>

<script>
export default {
  props: ['items', 'totalDays', 'noLabels', 'summaryLabel'],

  data() {
    return {
    }
  },

  computed: {
    stats() {
      const stats = { indicators: {}, totalValue: 0 }

      const fromDate = Math.min(...this.items.map(i => i.startDate));
      const toDate = Math.max(...this.items.map(i => i.endDate)) + 86300 * 1000;
      stats.totalDays = this.totalDays || Math.floor((toDate - fromDate) / 86400 / 1000) + 0;

      for (let item of this.items) {
        const indicatorStats = stats.indicators[item.indicator];
        if (!indicatorStats) indicatorStats = {}

        indicatorStats.color = item.color;
        indicatorStats.total = indicatorStats.total ? indicatorStats.total + 1 : 1;
        stats.totalValue++;
        stats.indicators[item.indicator] = indicatorStats;
      }

      // sort
      const namesSorted = Object.keys(stats.indicators).sort();
      const indicatorsSorted = {};
      for (let indicator of namesSorted) {
        indicatorsSorted[indicator] = stats.indicators[indicator]
      }
      stats.indicators = indicatorsSorted

      return stats;
    }
  },

  methods: {
  },
}
</script>

<style>
.stats {
  padding: 0 0 30px;
  text-align: left;
}
.stats-total {
  font-weight: bold;
  margin-bottom: 10px;
}
</style>
