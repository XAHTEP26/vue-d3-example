<template>
  <g
    fill="none"
    font-size="10"
    font-family="sans-serif"
    text-anchor="end"
  >
    <path stroke="currentColor" :d="d"></path>
    <g
      v-for="tick in ticks"
      :key="tick.position"
      :transform="transform(tick.position)"
    >
      <line stroke="currentColor" x2="-6"></line>
      <text fill="currentColor" dy="0.32em" x="-9">{{ tick.name }}</text>
    </g>
  </g>
</template>

<script>
export default {
  props: {
    range: {
      type: Array,
      required: true,
    },
    ticks: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    d() {
      return `M-6,${this.range[0] + 0.5}H0.5V${this.range[1] + 0.5}H-6`;
    },
  },
  methods: {
    transform(position) {
      return `translate(0,${position + 0.5})`;
    },
  },
};
</script>
