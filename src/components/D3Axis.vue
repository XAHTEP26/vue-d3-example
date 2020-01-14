<script>
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

export default ({
  data,
  props: {
    scale,
    orient,
    tickArguments = [],
    tickValues = null,
    tickFormat = null,
    tickSizeInner = 6,
    tickSizeOuter = 6,
    tickPadding = 3,
  },
}) => {
  const k = orient === top || orient === left ? -1 : 1;
  const x = orient === left || orient === right ? 'x' : 'y';
  const transform = orient === top || orient === bottom ? translateX : translateY;

  const values = tickValues === null ? (scale.ticks ? scale.ticks(...tickArguments) : scale.domain()) : tickValues;
  const format = tickFormat === null ? (scale.tickFormat ? scale.tickFormat(...tickArguments) : identity) : tickFormat;
  const spacing = Math.max(tickSizeInner, 0) + tickPadding;
  const range = scale.range();
  const range0 = +range[0] + 0.5;
  const range1 = +range[range.length - 1] + 0.5;
  const position = (scale.bandwidth ? center : number)(scale.copy());

  const tickTextAnchor = orient === left ? 'end' : 'middle';
  const d = orient === left || orient === right
    ? (tickSizeOuter ? `M${k * tickSizeOuter},${range0}H0.5V${range1}H${k * tickSizeOuter}` : `M0.5,${range0}V${range1}`)
    : (tickSizeOuter ? `M${range0},${k * tickSizeOuter}V0.5H${range1}V${k * tickSizeOuter}` : `M${range0},0.5H${range1}`);
  const tickDeltaY = orient === top ? '0em' : orient === bottom ? '0.71em' : '0.32em';
  const lineAttributes = {
    stroke: 'currentColor',
    [`${x}2`]: k * tickSizeInner,
  };
  const textAttributes = {
    fill: 'currentColor',
    dy: tickDeltaY,
    [x]: k * spacing,
  };

  return (
      <g
        fill="none"
        font-size="10"
        font-family="sans-serif"
        text-anchor={tickTextAnchor}
        {...data}
      >
        <path class="domain" stroke="currentColor" d={d}/>
        <transition-group name="ticks" tag="g">
          {values.map((value, index) => (
            <g
              key={index}
              class="tick"
              opacity="1"
              transform={transform(position(value))}
            >
              <line {...{ attrs: lineAttributes }}/>
              <text {...{ attrs: textAttributes }}>{format(value)}</text>
            </g>
          ))}
        </transition-group>
      </g>
  );
};
</script>

<style>
.ticks-enter-active,
.ticks-leave-active {
  transition: opacity .5s;
}

.ticks-enter,
.ticks-leave-to {
  opacity: 0;
}
</style>
