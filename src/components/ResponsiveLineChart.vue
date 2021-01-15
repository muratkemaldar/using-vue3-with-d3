<template>
  <div ref="resizeRef">
    <svg ref="svgRef">
      <g class="x-axis" />
      <g class="y-axis" />
    </svg>
  </div>
</template>

<script>
import { onMounted, ref, watchEffect } from "vue";
import {
  select,
  line,
  scaleLinear,
  min,
  max,
  curveBasis,
  axisBottom,
  axisLeft,
} from "d3";
import useResizeObserver from "@/use/resizeObserver";

export default {
  name: "ResponsiveLineChart",
  props: ["data"],
  setup(props) {
    // create ref to pass to D3 for DOM manipulation
    const svgRef = ref(null);

    // create another ref to observe resizing, since observing SVGs doesn't work!
    const { resizeRef, resizeState } = useResizeObserver();

    onMounted(() => {
      // pass ref with DOM element to D3, when mounted (DOM available)
      const svg = select(svgRef.value);

      // whenever any dependencies (like data, resizeState) change, call this!
      watchEffect(() => {
        const { width, height } = resizeState.dimensions;

        // scales: map index / data values to pixel values on x-axis / y-axis
        const xScale = scaleLinear()
          .domain([0, props.data.length - 1])
          .range([0, width]);

        const yScale = scaleLinear()
          .domain([min(props.data), max(props.data)])
          .range([height, 0]);

        // line generator: D3 method to transform an array of values to data points ("d") for a path element
        const lineGen = line()
          .curve(curveBasis)
          .x((value, index) => xScale(index))
          .y((value) => yScale(value));

        // render path element with D3's General Update Pattern
        svg
          .selectAll(".line")
          .data([props.data]) // pass entire data array
          .join("path")
          .attr("class", "line")
          .attr("stroke", "green")
          .attr("d", lineGen);

        // render axes with help of scales
        const xAxis = axisBottom(xScale);
        svg
          .select(".x-axis")
          .style("transform", `translateY(${height}px)`) // position on the bottom
          .call(xAxis);

        const yAxis = axisLeft(yScale);
        svg.select(".y-axis").call(yAxis);
      });
    });

    return { svgRef, resizeRef };
  },
};
</script>
