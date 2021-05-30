<template>
  <div :class="['stats-item', 'stats-item_' + validType]">
    <span v-if="color" class="stats-item-color" :style="'background-color: '+ color"></span>
    <span v-if="indicator" class="indicator-label">{{ indicator }}:</span>
    <span :class="['indicator-value', 'indicator_valid_' + validType]" :style="'color: ' + validColor">
      <span v-if="isSummary">{{ percent }}%</span>
      <span v-else>
        {{ percent }}%&nbsp;&nbsp;&nbsp;
        <span class="stats-item-counters">{{ indicatorStats.total }} / {{ totalDays }}</span>
      </span>
    </span>
  </div>
</template>

<script>
const successFrom = 80
const warningFrom = 33

export default {
  props: ['indicator', 'indicatorStats', 'totalDays', 'color', 'isSummary'],

  computed: {
    percent() {
      return Math.floor(this.indicatorStats.total / this.totalDays * 100);
    },
    validType() {
      if (this.percent < warningFrom) return 'danger';
      if (this.percent < successFrom) return 'warning';
      return 'success';
    },
    validColor() {
      let h
      /* if (this.percent < warningFrom) {
        h = Math.round(this.percent / warningFrom / 60)
      }
      else if (this.percent < successFrom) {
        h = 60 + Math.round(this.percent - warningFrom / 60)
      }
      else {
        h = 120
      } */
      h = this.percent / 100 * 120
      const color = `hsl(${h}, 40%, 50%)`
      return color
    }
  },

  methods: {
  },
}
</script>

<style>
.stats-item-color {
  display: inline-block;
  width: 20px;
  height: 9px;
}
.indicator-label {
  display: inline-block;
  min-width: 320px;
}
.indicator_valid_success {
  color: #67c23a;
}
.indicator_valid_warning {
  color: #e6a23c;
}
.indicator_valid_danger {
  color: #f56c6c;
}
</style>
