<template>
  <div id="app">
    <div>
      <button @click="update(data1)">Dataset 1</button>
      <button @click="update(data2)">Dataset 2</button>
    </div>

    <svg :width="width" :height="height">
      <d3-axis
        v-if="scaleX"
        orient="bottom"
        :scale="scaleX"
        :transform="`translate(${margin.left},${height - margin.bottom})`"
      />
      <d3-axis
        v-if="scaleY"
        orient="left"
        :scale="scaleY"
        :transform="`translate(${margin.left},${margin.top})`"
      />
      <path
        fill="none"
        stroke="steelblue"
        stroke-width="2.5"
        :d="tweenedPath"
        :transform="`translate(${margin.left},${margin.top})`"
      ></path>
    </svg>
  </div>
</template>

<script>
import {
  scaleLinear, max, interpolate,
} from 'd3';
import D3Axis from './components/D3Axis.vue';

function animate({ timing, draw, duration }) {
  const start = performance.now();

  requestAnimationFrame(function drawFrame(time) {
    // timeFraction изменяется от 0 до 1
    let timeFraction = (time - start) / duration;
    if (timeFraction > 1) timeFraction = 1;

    // вычисление текущего состояния анимации
    const progress = timing(timeFraction);

    draw(progress); // отрисовать её

    if (timeFraction < 1) {
      requestAnimationFrame(drawFrame);
    }
  });
}

export default {
  components: {
    D3Axis,
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
      path: '',
      tweenedPath: '',
      domainX: [0, 0],
      tweenedDomainX: [0, 0],
      data: [],
      data1: [
        { ser1: 0.3, ser2: 4 },
        { ser1: 2, ser2: 16 },
        { ser1: 3, ser2: 8 },
      ],
      data2: [
        { ser1: 1, ser2: 7 },
        { ser1: 4, ser2: 1 },
        { ser1: 6, ser2: 8 },
      ],
    };
  },
  watch: {
    data(newValue, oldValue) {
      // if (this.scaleX && this.scaleY) return '';
      /* const lineBuilder = line()
        .x(d => this.scaleX(d.ser1))
        .y(d => this.scaleY(d.ser2));
      const oldPath = lineBuilder(oldValue);
      const newPath = lineBuilder(newValue);
      const pathInterpolator = interpolate(oldPath, newPath); */

      const oldDomainX = [0, max(oldValue, d => d.ser1)];
      const newDomainX = [0, max(newValue, d => d.ser1)];
      const domainXInterpolator = interpolate(oldDomainX, newDomainX);

      animate({
        duration: 3000,
        timing: d => d,
        draw: (progress) => {
          // this.tweenedPath = pathInterpolator(progress);
          this.tweenedDomainX = domainXInterpolator(progress);
        },
      });
    },
  },
  computed: {
    scaleX() {
      return scaleLinear()
        .range([0, this.width - this.margin.left - this.margin.right])
        .domain(this.tweenedDomainX);
    },
    scaleY() {
      return scaleLinear()
        .range([this.height - this.margin.top - this.margin.bottom, 0])
        .domain([0, max(this.data, d => d.ser2)]);
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
