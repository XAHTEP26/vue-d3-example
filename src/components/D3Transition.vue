<script>
/* import {
    scaleLinear, line, max, interpolate,
  } from 'd3'; */

import * as d3 from 'd3';

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
  props: {
    data: {
      required: true,
    },
    duration: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      tweenedData: null,
    };
  },
  watch: {
    data: {
      handler(newData, oldData) {
        if (!this.duration) {
          this.tweenedData = newData;
          return;
        }

        const interpolator = d3.interpolate(oldData, newData);
        animate({
          duration: this.duration,
          timing: d => d,
          draw: (progress) => {
            this.tweenedData = interpolator(progress);
          },
        });
      },
      immediate: true,
    },
  },
  render() {
    return this.$scopedSlots.default(this.tweenedData);
  },
};
</script>
