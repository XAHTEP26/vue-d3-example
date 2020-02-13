<template>
  <div id="app">
    <div>
      <button @click="update()">Update</button>
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
        stroke="red"
        stroke-width="2.5"
        :d="tweenedPath"
        :transform="`translate(${margin.left},${margin.top})`"
      />
    </svg>
  </div>
</template>

<script>
import * as d3 from 'd3';
import D3AxisBottom from './components/D3AxisBottom.vue';
import D3AxisLeft from './components/D3AxisLeft.vue';

function generateFakeData() {
  let prev = 100;
  return new Array(200).fill(0).map((_, i) => {
    const y = prev + Math.random() * (Math.random() > 0.5 ? 1 : -1);
    prev = y;
    return {
      x: i,
      y,
    };
  });
}

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
      width: 1900,
      height: 400,
      data: [],
      tweenedPath: '',
    };
  },
  computed: {
    scaleX() {
      return d3.scaleLinear()
        .range([0, this.width - this.margin.left - this.margin.right])
        .domain(d3.extent(this.data, d => d.x));
    },
    scaleY() {
      return d3.scaleLinear()
        .range([this.height - this.margin.top - this.margin.bottom, 0])
        .domain(d3.extent(this.data, d => d.y));
    },
    path() {
      return this.lineBuilder(this.data);
    },
  },
  watch: {
    path: {
      handler(newPath) {
        const interpolator = d3.interpolate(this.$data.tweenedPath, newPath);
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
    update() {
      this.data = generateFakeData();
    },
  },
  beforeCreate() {
    this.lineBuilder = d3.line()
      .x(d => this.scaleX(d.x))
      .y(d => this.scaleY(d.y));
  },
  mounted() {
    this.update();
  },
};
</script>
