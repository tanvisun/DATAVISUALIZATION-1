<template>
  <!-- Placeholder now wraps the entire chart area -->
  <div class="chart-placeholder">
    <div class="form-group mb-3">
      <label for="countrySelect">Select Country</label>
      <select
        id="countrySelect"
        v-model="selectedCountry"
        @change="updateChart"
        class="form-control dropdown-small"
      >
        <option v-for="country in countries" :value="country" :key="country">
          {{ country }}
        </option>
      </select>
    </div>
    <div ref="chart"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "DonutChart",
  data() {
    return {
      countriesData: [],
      countries: [],
      selectedCountry: null,
      margin: { top: 200, right: 20, bottom: 40, left: 60 },
      width: 450,
      height: 250,
      svg: null,
      radius: null
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      d3.csv('/DATAVISUALIZATION-1/data/worlddata.csv')
        .then(data => {
          // Process columns: remove "%" and convert to float.
          data.forEach(d => {
            d["Tax revenue (%)"] =
              parseFloat(d["Tax revenue (%)"].replace("%", "").trim()) || 0;
            d["Total tax rate"] =
              parseFloat(d["Total tax rate"].replace("%", "").trim()) || 0;
          });
          // Use only the first 5 countries.
          this.countriesData = data.slice(0, 5);
          this.countries = this.countriesData.map(d => d.Country);
          this.selectedCountry = this.countries[0];
          this.createChart();
          this.updateChart();
        })
        .catch(error => {
          console.error("Error loading CSV data:", error);
        });
    },
    createChart() {
      // Clear any existing chart.
      d3.select(this.$refs.chart).selectAll("*").remove();
      
      // Determine the radius.
      this.radius = Math.min(this.width, this.height) / 2;
      
      // Create the SVG container and center the donut chart.
      this.svg = d3.select(this.$refs.chart)
        .append("svg")
        .attr("width", this.width + this.margin.left + this.margin.right)
        .attr("height", this.height + this.margin.top + this.margin.bottom)
        .append("g")
        .attr("transform", `translate(${(this.width + this.margin.left + this.margin.right) / 2}, ${(this.height + this.margin.top + this.margin.bottom) / 2})`);
    },
    updateChart() {
      // Remove existing arcs and labels.
      this.svg.selectAll("path").remove();
      this.svg.selectAll("text.arc-label").remove();

      // Remove any existing tooltip.
      d3.select("body").selectAll(".tooltip").remove();
      // Create tooltip.
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

      // Get data for the selected country.
      const countryData = this.countriesData.find(d => d.Country === this.selectedCountry);
      if (!countryData) return;

      // Build the data array for the donut.
      const rawData = [
        { key: "Tax revenue (%)", value: countryData["Tax revenue (%)"] },
        { key: "Total tax rate", value: countryData["Total tax rate"] }
      ];

      // Normalize values so they sum to 100.
      const total = rawData.reduce((sum, d) => sum + d.value, 0);
      const donutData = rawData.map(d => ({
        key: d.key,
        value: total === 0 ? 0 : (d.value / total) * 100,
        raw: d.value
      }));

      // Create the pie layout.
      const pie = d3.pie()
        .value(d => d.value)
        .sort(null);
      const data_ready = pie(donutData);

      // Create arc generator.
      const arc = d3.arc()
        .innerRadius(this.radius * 0.5)
        .outerRadius(this.radius * 1);

      // Define a color scale.
      const color = d3.scaleOrdinal()
        .domain(["Tax revenue (%)", "Total tax rate"])
        .range(["#953553", "#e7b4aa"]);

      // Mapping for legend and tooltip labels.
      const legendLabels = {
        "Tax revenue (%)": "Tax revenue%",
        "Total tax rate": "Total revenue"
      };

      // Draw arcs with hover effects and tooltips.
      this.svg.selectAll('path')
        .data(data_ready)
        .enter()
        .append('path')
        .attr('d', arc)
        .attr('fill', d => color(d.data.key))
        .attr("stroke", "white")
        .style("stroke-width", "2px")
        .on("mouseover", function(event, d) {
          d3.select(this)
            .transition()
            .duration(200)
            .attr("fill-opacity", 0.7);
          tooltip.style("display", "block")
                 .html(`<strong>${legendLabels[d.data.key]}</strong><br>Value: ${d.data.raw}`);
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
        })
        .transition()
        .duration(500)
        .attrTween('d', function(d) {
          const interpolate = d3.interpolate(d.startAngle, d.endAngle);
          return function(t) {
            d.endAngle = interpolate(t);
            return arc(d);
          };
        });

      // Add text labels on arcs for the percentage value.
      this.svg.selectAll('text.arc-label')
        .data(data_ready)
        .enter()
        .append('text')
        .attr("class", "arc-label")
        .attr("transform", d => "translate(" + arc.centroid(d) + ")")
        .attr("dy", "0.35em")
        .attr("text-anchor", "middle")
        .text(d => d.data.value.toFixed(1) + "%")
        .style("fill", "#fff")
        .style("font-size", "10px");

      // Create the legend in a separate group appended to the outer SVG.
      const outerSVG = d3.select(this.$refs.chart).select("svg");
      // Remove any existing legend.
      outerSVG.select(".legendGroup").remove();
      // Append a legend group to the right of the donut chart.
      const legendGroup = outerSVG.append("g")
        .attr("class", "legendGroup")
        .attr("transform", `translate(${this.width + this.margin.left - 80}, ${this.margin.top})`);

      const legend = legendGroup.selectAll(".legend")
        .data(donutData)
        .enter()
        .append("g")
        .attr("class", "legend")
        .attr("transform", (d, i) => `translate(0, ${i * 20})`);

      legend.append("rect")
        .attr("width", 12)
        .attr("height", 12)
        .attr("fill", d => color(d.key));

      legend.append("text")
        .attr("x", 16)
        .attr("y", 12)
        .text(d => {
          if (d.key === "Tax revenue (%)") return "Tax revenue% (blue)";
          if (d.key === "Total tax rate") return "Total revenue (orange)";
          return d.key;
        })
        .attr("font-size", "12px")
        .attr("fill", "#000");
    }
  }
};
</script>

<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;
  border: 1px dashed #ccc;
  padding: 10px;
}
.tooltip {
  box-shadow: 0px 0px 5px rgba(0,0,0,0.3);
}
</style>
