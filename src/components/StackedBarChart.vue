<template>
  <div>
    <!-- Chart container with placeholder background -->
    <div ref="chart" class="chart-placeholder"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "StackedBarChart",
  data() {
    return {
      countriesData: [],
      // Updated margins: increased top to leave space for title
      margin: { top: 200, right: 63, bottom: 40, left: 60 },
      // Reduced chart size
      width: 400,
      height: 250,
      svg: null
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      d3.csv('/data/worlddata.csv')
        .then(data => {
          // Process the percentage columns: remove "%" and convert to float.
          data.forEach(d => {
            d["Gross primary education enrollment (%)"] =
              parseFloat(d["Gross primary education enrollment (%)"].replace("%", "").trim()) || 0;
            d["Gross tertiary education enrollment (%)"] =
              parseFloat(d["Gross tertiary education enrollment (%)"].replace("%", "").trim()) || 0;
          });
          // Use only the first 5 countries.
          this.countriesData = data.slice(0, 5);
          this.createChart();
          this.updateChart();
        })
        .catch(error => {
          console.error("Error loading CSV data:", error);
        });
    },
    createChart() {
      // Clear any previous content.
      d3.select(this.$refs.chart).selectAll("*").remove();

      // Create the outer SVG container.
      const svgContainer = d3.select(this.$refs.chart)
        .append("svg")
        .attr("width", this.width + this.margin.left + this.margin.right)
        .attr("height", this.height + this.margin.top + this.margin.bottom);
      
      // Append title in the top margin area.
      svgContainer.append("text")
        .attr("x", (this.width + this.margin.left + this.margin.right) / 2)
        .attr("y", this.margin.top / 2)
        .attr("text-anchor", "middle")
        .style("font-size", "16px")
        .style("font-weight", "bold")
        .text("Stacked Bar Chart Title");
      
      // Create a group for the chart area.
      this.svg = svgContainer.append("g")
        .attr("transform", `translate(${this.margin.left},${this.margin.top})`);
    },
    updateChart() {
      // Remove any existing tooltip.
      d3.select("body").selectAll(".tooltip").remove();
      // Append tooltip div to the body.
      const tooltip = d3.select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background", "lightsteelblue")
        .style("padding", "5px")
        .style("border-radius", "3px")
        .style("pointer-events", "none")
        .style("font-size", "12px")
        .style("display", "none");

      // Define the indicators to stack.
      const keys = [
        "Gross primary education enrollment (%)",
        "Gross tertiary education enrollment (%)"
      ];

      // Compute the maximum total value for any country.
      const maxTotal = d3.max(this.countriesData, d =>
        keys.reduce((sum, key) => sum + d[key], 0)
      );

      // Create scales.
      const xScale = d3.scaleLinear()
        .domain([0, maxTotal])
        .range([0, this.width]);

      const yScale = d3.scaleBand()
        .domain(this.countriesData.map(d => d.Country))
        .range([0, this.height])
        .padding(0.1);

      // Color scale for the two series.
      const color = d3.scaleOrdinal()
        .domain(keys)
        .range(["#953553", "#e7b4aa"]);

      // Use d3.stack() to compute the stacked data.
      const stackGenerator = d3.stack().keys(keys);
      const stackedData = stackGenerator(this.countriesData);

      // Draw the stacked bars.
      this.svg.selectAll(".serie")
        .data(stackedData)
        .enter().append("g")
        .attr("class", "serie")
        .attr("fill", d => color(d.key))
        .selectAll("rect")
        .data(d => d.map(p => {
          p.indicator = d.key; 
          return p;
        }))
        .enter().append("rect")
        .attr("x", d => xScale(d[0]))
        .attr("y", d => yScale(d.data.Country))
        .attr("width", d => xScale(d[1]) - xScale(d[0]))
        .attr("height", yScale.bandwidth())
        // Hover tooltip.
        .on("mouseover", function(event, d) {
          d3.select(this)
            .transition()
            .duration(200)
            .attr("fill-opacity", 0.7);
          tooltip.style("display", "block")
                 .html(`<strong>${d.data.Country}</strong><br>${d.indicator}: ${d.data[d.indicator]}`);
        })
        .on("mousemove", function(event) {
          tooltip.style("left", (event.pageX + 10) + "px")
                 .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", function() {
          d3.select(this)
            .transition()
            .duration(200)
            .attr("fill-opacity", 1);
          tooltip.style("display", "none");
        });

      // Add x-axis.
      const xAxis = d3.axisBottom(xScale);
      this.svg.append("g")
        .attr("transform", `translate(0, ${this.height})`)
        .call(xAxis);

      // Add y-axis.
      const yAxis = d3.axisLeft(yScale);
      this.svg.append("g")
        .call(yAxis);

      // ----- PLACE LEGEND IN THE TOP-RIGHT CORNER -----
      // Create a group for the legend and position it near the top-right.
      const legendGroup = this.svg.append("g")
        .attr("class", "legendGroup")
        // Shift it slightly left of the chart's right edge and near the top
        .attr("transform", `translate(${this.width - 100}, 0)`);

      // Create one 'g' per key in 'keys'.
      const legend = legendGroup.selectAll(".legend")
        .data(keys)
        .enter().append("g")
        .attr("class", "legend")
        // Space them vertically (20px apart).
        .attr("transform", (d, i) => `translate(0, ${i * 20})`);

      // Draw the color swatch.
      legend.append("rect")
        .attr("x", 0)
        .attr("y", 0)
        .attr("width", 18)
        .attr("height", 18)
        .style("fill", d => color(d));

      // Add the text label to the right of the swatch.
      legend.append("text")
        .attr("x", 25)
        .attr("y", 14)
        .style("text-anchor", "start")
        .style("font-size", "12px")
        .text(d => d);
    }
  }
};
</script>

<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;
  border: 1px dashed #ccc;
  padding: 10px;
  min-height: 590px;
}

.tooltip {
  box-shadow: 0px 0px 5px rgba(0,0,0,0.3);
}
</style>
