<template>
  <g
    fill="none"
    font-size="10"
    font-family="sans-serif"
    text-anchor="end"
  >
    <path stroke="currentColor" :d="d"></path>
    <transition-group name="tick" tag="g">
      <g
        class="tick"
        v-for="tick in ticks"
        :key="tick"
        :transform="transform(tick)"
      >
        <line stroke="currentColor" x2="-6"></line>
        <text fill="currentColor" dy="0.32em" x="-9">{{ tick }}</text>
      </g>
    </transition-group>
  </g>
</template>

<script>
export default {
  props: {
    scale: {
      type: Function,
      required: true,
    },
  },
  computed: {
    range() {
      return this.scale.range();
    },
    ticks() {
      return this.scale.ticks();
    },
    d() {
      return `M-6,${this.range[0] + 0.5}H0.5V${this.range[1] + 0.5}H-6`;
    },
  },
  methods: {
    transform(tick) {
      return `translate(0,${this.scale(tick) + 0.5})`;
    },
  },
};
</script>

<style scoped lang="scss">
.tick {
  transition: .25s;

  &-enter,
  &-leave-to {
    opacity: 0;
  }
}
</style>
