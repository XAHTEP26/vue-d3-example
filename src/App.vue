<template>
  <div id="app">
    <div>
      <button @click="update(data1)">Dataset 1</button>
      <button @click="update(data2)">Dataset 2</button>
    </div>

    <svg :width="width" :height="height">
      <d3-axis-bottom
        :scale="scaleX"
        :transform="`translate(${margin.left},${height - margin.bottom})`"
      />
      <d3-axis-left
        :scale="scaleY"
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
import * as d3 from 'd3';
import { interpolatePath } from 'd3-interpolate-path';
import D3AxisBottom from './components/D3AxisBottom.vue';
import D3AxisLeft from './components/D3AxisLeft.vue';

function onFrame({
  draw,
  duration = 250,
  ease = d3.easeCubic,
  delay = 0,
}) {
  const timer = d3.timer((elapsed) => {
    const timeFraction = Math.min(elapsed / duration, 1);
    const progress = ease(timeFraction);
    if (progress === 1) timer.stop();
    draw(progress);
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
        { ser1: 1, ser2: 6 },
        { ser1: 2, ser2: 10 },
        { ser1: 3, ser2: 8 },
        { ser1: 4, ser2: 9 },
        { ser1: 5, ser2: 8 },
      ],
      data2: [
        { ser1: 3, ser2: 8 },
        { ser1: 3.5, ser2: 8.7 },
        { ser1: 4, ser2: 9 },
        { ser1: 4.5, ser2: 10.3 },
        { ser1: 5, ser2: 12 },
        { ser1: 5.5, ser2: 14 },
        { ser1: 6, ser2: 17 },
      ],
      tweenedPath: '',
    };
  },
  computed: {
    scaleX() {
      return d3.scaleLinear()
        .range([0, this.width - this.margin.left - this.margin.right])
        .domain(d3.extent(this.data, d => d.ser1));
    },
    scaleY() {
      return d3.scaleLinear()
        .range([this.height - this.margin.top - this.margin.bottom, 0])
        .domain(d3.extent(this.data, d => d.ser2));
    },
    path() {
      const lineBuilder = d3.line()
        .x(d => this.scaleX(d.ser1))
        .y(d => this.scaleY(d.ser2));
      return lineBuilder(this.data);
    },
  },
  watch: {
    path: {
      handler(newPath) {
        const interpolator = interpolatePath(this.$data.tweenedPath, newPath);
        onFrame({
          draw: (progress) => {
            this.$data.tweenedPath = interpolator(progress);
          },
        });
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
