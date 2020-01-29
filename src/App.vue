<template>
  <div id="app">
    <div>
      <button @click="update(data1)">Dataset 1</button>
      <button @click="update(data2)">Dataset 2</button>
      <button @click="switchAxis()">Switch</button>
    </div>

    <svg :width="width" :height="height">
      <d3-axis
        v-if="scaleX"
        :orient="orientX"
        :scale="scaleX"
        :transform="`translate(${margin.left},${height - margin.bottom})`"
      />
      <d3-axis
        v-if="scaleY"
        :orient="orientY"
        :scale="scaleY"
        :transform="`translate(${margin.left},${margin.top})`"
      />
      <d3-transition :data="path" :duration="3000" v-slot:default="tweenedPath">
        <path
          fill="none"
          stroke="steelblue"
          stroke-width="2.5"
          :d="tweenedPath"
          :transform="`translate(${margin.left},${margin.top})`"
        />
      </d3-transition>
    </svg>
  </div>
</template>

<script>
import {
  scaleLinear, line, max,
} from 'd3';
import D3Axis from './components/D3Axis.vue';
import D3Transition from './components/D3Transition.vue';

export default {
  components: {
    D3Axis,
    D3Transition,
  },
  data() {
    return {
      margin: {
        top: 10,
        right: 30,
        bottom: 30,
        left: 50,
      },
      orientX: 'bottom',
      orientY: 'left',
      width: 460,
      height: 400,
      data: [],
      data1: [
        { ser1: 0.3, ser2: 4 },
        { ser1: 2, ser2: 16 },
        { ser1: 3, ser2: 8 },
      ],
      data2: [{ ser1: 1, ser2: 7 }, { ser1: 4, ser2: 1 }, { ser1: 6, ser2: 8 }],
    };
  },
  computed: {
    scaleX() {
      return scaleLinear()
        .range([0, this.width - this.margin.left - this.margin.right])
        .domain([0, max(this.data, d => d.ser1)]);
    },
    scaleY() {
      return scaleLinear()
        .range([this.height - this.margin.top - this.margin.bottom, 0])
        .domain([0, max(this.data, d => d.ser2)]);
    },
    path() {
      const lineBuilder = line()
        .x(d => this.scaleX(d.ser1))
        .y(d => this.scaleY(d.ser2));
      return lineBuilder(this.data);
    },
  },
  methods: {
    update(data) {
      this.data = data;
    },
    switchAxis() {
      [this.orientX, this.orientY] = [this.orientY, this.orientX];
    },
  },
  mounted() {
    this.update(this.data1);
  },
};
</script>
