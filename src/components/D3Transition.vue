<script>
import * as d3 from 'd3';

export default {
  props: {
    data: {
      required: true,
    },
    duration: {
      type: Number,
      default: 250,
    },
    delay: {
      type: Number,
      default: 0,
    },
    ease: {
      type: Function,
      default: d3.easeCubic,
    },
  },
  data() {
    return {
      tweenedData: null,
    };
  },
  watch: {
    data: {
      handler(newData) {
        if (!this.duration) {
          this.tweenedData = newData;
          return;
        }

        const interpolator = d3.interpolate(this.tweenedData, newData);

        const timer = d3.timer((elapsed) => {
          let timeFraction = elapsed / this.duration;
          if (timeFraction < 0) timeFraction = 0;
          if (timeFraction > 1) timeFraction = 1;
          const progress = this.ease(timeFraction);
          this.$set(this.$data, 'tweenedData', interpolator(progress));
          if (progress === 1) timer.stop();
        }, this.delay);
      },
      immediate: true,
    },
  },
  render() {
    return this.$scopedSlots.default(this.tweenedData);
  },
};
</script>
