<template>
  <div id="app">
    <div>
      <button @click="update(data1)">Dataset 1</button>
      <button @click="update(data2)">Dataset 2</button>
    </div>

    <svg :width="width" :height="height">
      <d3-axis-bottom
        :range="rangeX"
        :ticks="tweenedTicksX"
        :transform="`translate(${margin.left},${height - margin.bottom})`"
      />
      <d3-axis-left
        :range="rangeY"
        :ticks="tweenedTicksY"
        :transform="`translate(${margin.left},${margin.top})`"
      />
      <path
        fill="none"
        stroke="steelblue"
        stroke-width="2.5"
        :d="tweenedPath"
        :transform="`translate(${margin.left},${margin.top})`"
      />
    </svg>
  </div>
</template>

<script>
import {
  scaleLinear, line, max,
  interpolate,
  timer,
  easeCubic,
} from 'd3';
import D3AxisBottom from './components/D3AxisBottom.vue';
import D3AxisLeft from './components/D3AxisLeft.vue';

function animate(from, to, {
  duration = 250,
  ease = easeCubic,
  delay = 0,
}) {
  const animatedProps = Object.keys(to);
  const animations = animatedProps.map(prop => ({
    interpolator: interpolate(from[prop], to[prop]),
    prop,
  }));

  const theTimer = timer((elapsed) => {
    let timeFraction = elapsed / duration;
    if (timeFraction < 0) timeFraction = 0;
    if (timeFraction > 1) timeFraction = 1;
    const progress = ease(timeFraction);
    animations.forEach((animation) => {
      /* eslint-disable */
      from[animation.prop] = animation.interpolator(progress);
      /* eslint-enable */
    });
    if (progress === 1) theTimer.stop();
  }, delay);
}

export default {
  components: {
    D3AxisBottom,
    D3AxisLeft,
  },
  data() {
    return {
      margin: {
        top: 10,
        right: 30,
        bottom: 30,
        left: 50,
      },
      width: 460,
      height: 400,
      data: [],
      data1: [
        { ser1: 0.3, ser2: 4 },
        { ser1: 2, ser2: 16 },
        { ser1: 3, ser2: 8 },
      ],
      data2: [{ ser1: 1, ser2: 7 }, { ser1: 4, ser2: 1 }, { ser1: 6, ser2: 9 }],
      tweenedPath: '',
      tweenedTicksX: [],
      tweenedTicksY: [],
    };
  },
  computed: {
    rangeX() {
      return [0, this.width - this.margin.left - this.margin.right];
    },
    scaleX() {
      return scaleLinear()
        .range(this.rangeX)
        .domain([0, max(this.data, d => d.ser1)]);
    },
    ticksX() {
      if (!this.scaleX) return [];
      return this.scaleX.ticks().map(tick => ({
        name: tick,
        position: this.scaleX(tick),
      }));
    },
    rangeY() {
      return [this.height - this.margin.top - this.margin.bottom, 0];
    },
    scaleY() {
      return scaleLinear()
        .range(this.rangeY)
        .domain([0, max(this.data, d => d.ser2)]);
    },
    ticksY() {
      if (!this.scaleY) return [];
      return this.scaleY.ticks().map(tick => ({
        name: tick,
        position: this.scaleY(tick),
      }));
    },
    path() {
      const lineBuilder = line()
        .x(d => this.scaleX(d.ser1))
        .y(d => this.scaleY(d.ser2));
      return lineBuilder(this.data);
    },
  },
  watch: {
    data: {
      handler() {
        animate(this.$data, { tweenedPath: this.path, tweenedTicksX: this.ticksX, tweenedTicksY: this.ticksY }, { duration: 500 });
      },
      immediate: true,
    },
  },
  methods: {
    update(data) {
      this.data = data;
    },
  },
  mounted() {
    this.update(this.data1);
  },
};
</script>
