<template>
  <!-- Entire chart area wrapped in a placeholder -->
  <div class="chart-placeholder">
    <div ref="chart"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "ParallelCoordinatesChart",
  data() {
    return {
      countriesData: [],
      // Define the four indicators to use in the chart.
      dimensions: ["Life expectancy", "Fertility Rate", "Infant mortality", "Population"],
      margin: { top: 30, right: 100, bottom: 10, left: -60 }, // increased right margin for legend
      width: 550,
      height: 400,
      svg: null,
      x: null,   // Ordinal scale for the x position of each axis.
      y: {}      // Object to store a y-scale for each dimension.
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      d3.csv('/data/worlddata.csv').then(data => {
        // Define the list of countries to include.
        const selectedCountries = new Set([
          "Afghanistan",
          "Albania",
          "Algeria",
          "Angola",
          "Antigua and Barbuda"
        ]);
        
        // Process and filter the data.
        data.forEach(d => {
          // Parse numeric fields for the selected indicators.
          d["Life expectancy"] = parseFloat(d["Life expectancy"]);
          d["Fertility Rate"] = parseFloat(d["Fertility Rate"]);
          d["Infant mortality"] = parseFloat(d["Infant mortality"]);
          // Remove commas from Population before converting to number.
          d["Population"] = parseFloat(d["Population"].replace(/,/g, ""));
        });
        // Filter the dataset to include only the selected countries.
        this.countriesData = data.filter(d => selectedCountries.has(d.Country));
        this.createChart();
      }).catch(error => console.error("Error loading CSV data:", error));
    },
    createChart() {
      // Clear any existing chart elements.
      d3.select(this.$refs.chart).selectAll("*").remove();

      // Create the outer SVG container.
      const outerSVG = d3.select(this.$refs.chart)
        .append("svg")
        .attr("width", this.width + this.margin.left + this.margin.right)
        .attr("height", this.height + this.margin.top + this.margin.bottom);

      // Save the outer container as svgContainer.
      const svgContainer = outerSVG;

      // Append a group for the chart and apply margins.
      this.svg = outerSVG.append("g")
        .attr("transform", `translate(${this.margin.left},${this.margin.top})`);

      // Create an ordinal scale to position each dimension along the x-axis.
      this.x = d3.scalePoint()
        .range([0, this.width])
        .padding(1)
        .domain(this.dimensions);

      // For each dimension, create a linear y-scale based on the extent of the data.
      this.dimensions.forEach(dim => {
        this.y[dim] = d3.scaleLinear()
          .domain(d3.extent(this.countriesData, d => +d[dim]))
          .range([this.height, 0]);
      });

      // Create a color scale for countries.
      const color = d3.scaleOrdinal()
        .domain(this.countriesData.map(d => d.Country))
        .range(d3.schemeReds[9].slice(4));

      // Create a tooltip element.
      const tooltip = d3.select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background", "rgba(0,0,0,0.7)")
        .style("color", "#fff")
        .style("padding", "5px 10px")
        .style("border-radius", "4px")
        .style("pointer-events", "none")
        .style("opacity", 0);

      // Draw a line (path) for each country connecting all dimensions.
      this.svg.selectAll("path")
        .data(this.countriesData)
        .enter()
        .append("path")
        .attr("d", d => this.path(d))
        .attr("stroke", d => color(d.Country))
        .attr("fill", "none")
        .attr("stroke-width", 3)
        .attr("opacity", 0.7)
        .each(function() {
          // Animate the line drawing using stroke-dasharray technique.
          const totalLength = this.getTotalLength();
          d3.select(this)
            .attr("stroke-dasharray", totalLength)
            .attr("stroke-dashoffset", totalLength)
            .transition()
            .duration(1500)
            .attr("stroke-dashoffset", 0);
        })
        .on("mouseover", function(event, d) {
          d3.select(this)
            .transition()
            .duration(200)
            .attr("stroke-width", 7)
            .attr("opacity", 1);
          tooltip.transition()
            .duration(200)
            .style("opacity", 0.9);
          tooltip.html(
            `<strong>${d.Country}</strong><br/>
            Life expectancy: ${d["Life expectancy"]}<br/>
            Fertility Rate: ${d["Fertility Rate"]}<br/>
            Infant mortality: ${d["Infant mortality"]}<br/>
            Population: ${d["Population"]}`
          )
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
        })
        .on("mousemove", function(event) {
          tooltip.style("left", (event.pageX + 10) + "px")
                 .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", function() {
          d3.select(this)
            .transition()
            .duration(200)
            .attr("stroke-width", 1.5)
            .attr("opacity", 0.7);
          tooltip.transition()
            .duration(200)
            .style("opacity", 0);
        });

      // Draw the vertical axes for each dimension.
      this.dimensions.forEach(dim => {
        let g = this.svg.append("g")
          .attr("transform", `translate(${this.x(dim)})`);

        g.call(d3.axisLeft(this.y[dim]));

        // Add a title above each axis.
        g.append("text")
          .style("text-anchor", "middle")
          .attr("y", -9)
          .text(dim);
      });

      // Create the legend using the already declared svgContainer.
      const legendGroup = svgContainer.append("g")
        .attr("class", "legendGroup")
        .attr("transform", `translate(${this.width + this.margin.left + 20}, ${this.margin.top})`);

      const legend = legendGroup.selectAll(".legend")
        .data(this.countriesData)
        .enter()
        .append("g")
        .attr("class", "legend")
        .attr("transform", (d, i) => `translate(0, ${i * 20})`);

      legend.append("rect")
        .attr("width", 12)
        .attr("height", 12)
        .attr("fill", d => color(d.Country));

      legend.append("text")
        .attr("x", 16)
        .attr("y", 12)
        .text(d => d.Country)
        .attr("font-size", "12px")
        .attr("fill", "#000");
    },
    // Helper function that generates the path for a given country's data.
    path(d) {
      return d3.line()(this.dimensions.map(p => [this.x(p), this.y[p](d[p])]));
    }
  }
};
</script>
    
<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;
  border: 1px dashed #ccc;
  padding: 10px;
  min-height: 520px; /* Adjust as needed for Parallel Coordinates Chart */
}

.tooltip {
  font-family: Arial, sans-serif;
  font-size: 12px;
}
</style>
