<template>
  <div class="chart-placeholder">
    <div ref="chart"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: 'CirclePackingChart',
  data() {
    return {
      width: 200,
      height: 200,
    };
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      d3.csv('/data/worlddata.csv').then(data => {
        // Define the official languages to include manually.
        const validLanguages = ['Arabic', 'Portuguese', 'English', 'French', 'Korean', 'Spanish'];
        
        // Process all rows and filter for only the valid languages.
        const processed = data.map(d => ({
          language: d["Official language"] || "Unknown",
          country: d.Country,
          population: +d.Population.replace(/,/g, '') || 0
        })).filter(d => d.population > 0 && validLanguages.includes(d.language));

        const languageMap = d3.group(processed, d => d.language);
        const hierarchyData = {
          name: "World",
          children: Array.from(languageMap, ([language, countries]) => ({
            name: language,
            children: countries.map(c => ({
              name: c.country,
              value: c.population
            }))
          }))
        };

        const root = d3.hierarchy(hierarchyData)
          .sum(d => d.value)
          .sort((a, b) => b.value - a.value);

        const pack = d3.pack()
          .size([this.width, this.height])
          .padding(3);

        pack(root);

        const svg = d3.select(this.$refs.chart)
          .append("svg")
          .attr("viewBox", `0 0 ${this.width} ${this.height}`)
          .style("font", "10px sans-serif")
          .style("cursor", "pointer");

        // Use a custom ordinal scale that cycles between the two specified colours.
        const color = d3.scaleOrdinal().range(["#953553", "#e7b4aa"]);

        const tooltip = d3.select("body").append("div")
          .attr("class", "tooltip")
          .style("position", "absolute")
          .style("background", "rgba(0,0,0,0.8)")
          .style("color", "#fff")
          .style("padding", "6px 10px")
          .style("border-radius", "5px")
          .style("font-size", "12px")
          .style("pointer-events", "none")
          .style("opacity", 0);

        const g = svg.append("g")
          .attr("transform", `translate(${this.width / 2}, ${this.height / 2})`);

        let focus = root;
        let view;

        const node = g.selectAll("circle")
          .data(root.descendants())
          .enter().append("circle")
          // For nodes with children, use the custom color scale (cycling between the two colours).
          // For leaf nodes, still use "#ddd".
          .attr("fill", d => d.children ? color(d.depth) : "#ddd")
          .attr("pointer-events", d => d.children ? "all" : "none")
          .on("click", (event, d) => zoom(focus === d ? root : d))
          .on("mouseover", function(event, d) {
            if (!d.children) {
              tooltip.transition().duration(200).style("opacity", 0.9);
              tooltip.html(`<strong>${d.data.name}</strong><br>Population: ${d.data.value.toLocaleString()}`)
                .style("left", (event.pageX + 10) + "px")
                .style("top", (event.pageY - 28) + "px");
            }
          })
          .on("mousemove", function(event) {
            tooltip.style("left", (event.pageX + 10) + "px")
                   .style("top", (event.pageY - 28) + "px");
          })
          .on("mouseout", function() {
            tooltip.transition().duration(200).style("opacity", 0);
          });

        const label = g.selectAll("text")
          .data(root.descendants())
          .enter().append("text")
          .style("fill-opacity", d => d.parent === root ? 1 : 0)
          .style("display", d => d.parent === root ? "inline" : "none")
          .text(d => d.data.name)
          .attr("text-anchor", "middle")
          .style("pointer-events", "none");

        const zoomTo = v => {
          const k = this.width / v[2];
          view = v;
          label.attr("transform", d => `translate(${(d.x - v[0]) * k},${(d.y - v[1]) * k})`);
          node.attr("transform", d => `translate(${(d.x - v[0]) * k},${(d.y - v[1]) * k})`);
          node.attr("r", d => d.r * k);
        };

        const zoom = d => {
          focus = d;
          const transition = svg.transition()
            .duration(750)
            .tween("zoom", () => {
              const i = d3.interpolateZoom(view, [focus.x, focus.y, focus.r * 2]);
              return t => zoomTo(i(t));
            });

          label
            .transition(transition)
            .style("fill-opacity", d => d.parent === focus ? 1 : 0)
            .on("start", function() {
              if (this.__data__.parent === focus) d3.select(this).style("display", "inline");
            })
            .on("end", function() {
              if (this.__data__.parent !== focus) d3.select(this).style("display", "none");
            });
        };

        zoomTo([root.x, root.y, root.r * 2]);
      });
    }
  }
};
</script>

<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;
  border: 1px dashed #ccc;
  padding: 10px;
  min-height: 220px;
}
.tooltip {
  transition: opacity 0.3s ease;
  z-index: 10;
}
</style>
