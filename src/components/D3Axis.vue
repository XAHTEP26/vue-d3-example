<template>
  <g
    fill="none"
    font-size="10"
    font-family="sans-serif"
    :text-anchor="tickTextAnchor"
  >
    <d3-transition :data="d" :duration="3000" v-slot:default="tweenedD">
      <path class="domain" stroke="currentColor" :d="tweenedD"></path>
    </d3-transition>
    <g
      v-for="value in values"
      :key="value"
      class="tick"
      opacity="1"
      :transform="transform(position(value))"
    >
      <line stroke="currentColor" v-bind="{ [`${x}2`]: k * tickSizeInner }"></line>
      <text fill="currentColor" :dy="tickDeltaY" v-bind="{ [x]: k * spacing }">{{ format(value) }}</text>
    </g>
  </g>
</template>

<script>
import D3Transition from './D3Transition.vue';

const top = 'top';
const bottom = 'bottom';
const left = 'left';
const right = 'right';

function identity(d) {
  return d;
}

function translateX(x) {
  return `translate(${x + 0.5},0)`;
}

function translateY(y) {
  return `translate(0,${y + 0.5})`;
}

function number(scale) {
  return d => +scale(d);
}

function center(scale) {
  let offset = Math.max(0, scale.bandwidth() - 1) / 2; // Adjust for 0.5px offset.
  if (scale.round()) offset = Math.round(offset);
  return d => +scale(d) + offset;
}

export default {
  components: {
    D3Transition,
  },
  props: {
    scale: {
      type: Function,
      required: true,
    },
    orient: {
      type: String,
      validator(value) {
        return [top, bottom, left, right].includes(value);
      },
    },
    tickArguments: {
      type: Array,
      default: () => [],
    },
    tickValues: {
      type: Array,
      default: null,
    },
    tickFormat: {
      type: Function,
      default: null,
    },
    tickSizeInner: {
      type: Number,
      default: 6,
    },
    tickSizeOuter: {
      type: Number,
      default: 6,
    },
    tickPadding: {
      type: Number,
      default: 3,
    },
  },
  computed: {
    k() {
      return this.orient === top || this.orient === left ? -1 : 1;
    },
    x() {
      return this.orient === left || this.orient === right ? 'x' : 'y';
    },
    transform() {
      return this.orient === top || this.orient === bottom ? translateX : translateY;
    },
    values() {
      return this.tickValues === null ? (this.scale.ticks ? this.scale.ticks(...this.tickArguments) : this.scale.domain()) : this.tickValues;
    },
    format() {
      return this.tickFormat === null ? (this.scale.tickFormat ? this.scale.tickFormat(...this.tickArguments) : identity) : this.tickFormat;
    },
    spacing() {
      return Math.max(this.tickSizeInner, 0) + this.tickPadding;
    },
    position() {
      return (this.scale.bandwidth ? center : number)(this.scale.copy());
    },
    tickTextAnchor() {
      return this.orient === left ? 'end' : 'middle';
    },
    d() {
      const range = this.scale.range();
      const range0 = +range[0] + 0.5;
      const range1 = +range[range.length - 1] + 0.5;
      return this.orient === left || this.orient === right
        ? (this.tickSizeOuter ? `M${this.k * this.tickSizeOuter},${range0}H0.5V${range1}H${this.k * this.tickSizeOuter}` : `M0.5,${range0}V${range1}`)
        : (this.tickSizeOuter ? `M${range0},${this.k * this.tickSizeOuter}V0.5H${range1}V${this.k * this.tickSizeOuter}` : `M${range0},0.5H${range1}`);
    },
    tickDeltaY() {
      return this.orient === top ? '0em' : this.orient === bottom ? '0.71em' : '0.32em';
    },
  },
  watch: {
    scale: {
      handler() {},
      immediate: true,
    },
  },
};
</script>
