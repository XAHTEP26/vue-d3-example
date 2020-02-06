<template>
  <g
    fill="none"
    font-size="10"
    font-family="sans-serif"
    text-anchor="middle"
  >
    <path stroke="currentColor" :d="d"></path>
    <g
      v-for="tick in ticks"
      :key="tick.position"
      :transform="transform(tick.position)"
    >
      <line stroke="currentColor" y2="6"></line>
      <text fill="currentColor" dy="0.71em" y="9">{{ tick.name }}</text>
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
      return `M${this.range[0] + 0.5},6V0.5H${this.range[1] + 0.5}V6`;
    },
  },
  methods: {
    transform(position) {
      return `translate(${position + 0.5},0)`;
    },
  },
};
</script>
