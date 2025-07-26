<template>
  <div class="chart-placeholder">
    <div class="form-group row mb-3">
      <div class="col-md-6">
        <label for="countryA">Select Country A</label>
        <select v-model="selectedA" id="countryA" class="form-control">
          <option v-for="c in countries" :key="c" :value="c">{{ c }}</option>
        </select>
      </div>
      <div class="col-md-6">
        <label for="countryB">Select Country B</label>
        <select v-model="selectedB" id="countryB" class="form-control">
          <option v-for="c in countries" :key="c" :value="c">{{ c }}</option>
        </select>
      </div>
    </div>
    <div ref="chart"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "DifferenceChart",
  data() {
    return {
      data: [],
      countries: [],
      selectedA: null,
      selectedB: null,
      width: 600,
      height: 300,
      margin: { top: 30, right: 30, bottom: 40, left: 100 },
    };
  },
  watch: {
    selectedA() {
      this.updateChart();
    },
    selectedB() {
      this.updateChart();
    },
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      d3.csv('/DATAVISUALIZATION-1/data/worlddata.csv').then(data => {
        // Limit to first 10 countries
        const sliced = data.slice(0, 10);
  
        // Parse GDP (remove $ and commas)
        sliced.forEach(d => {
          const raw = d["GDP"].replace(/[$,]/g, '').trim();
          d.GDP = parseFloat(raw);
        });
  
        this.data = sliced;
        this.countries = sliced.map(d => d.Country);
        this.selectedA = this.countries[0];
        this.selectedB = this.countries[1];
        this.drawChart();
      });
    },
    drawChart() {
      const svg = d3.select(this.$refs.chart)
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);
  
      this.svg = svg;
      this.updateChart();
    },
    updateChart() {
      if (!this.selectedA || !this.selectedB || !this.svg) return;
  
      const svg = this.svg;
      svg.selectAll("*").remove();
  
      const dataA = this.data.find(d => d.Country === this.selectedA);
      const dataB = this.data.find(d => d.Country === this.selectedB);
  
      const gdpA = dataA?.GDP || 0;
      const gdpB = dataB?.GDP || 0;
  
      const maxVal = d3.max([gdpA, gdpB]) * 1.1;
  
      const x = d3.scaleLinear()
        .domain([0, maxVal])
        .range([this.margin.left, this.width - this.margin.right]);
  
      const y = d3.scaleBand()
        .domain([this.selectedA, this.selectedB])
        .range([this.margin.top, this.height - this.margin.bottom])
        .padding(0.4);
  
      // Tooltip setup
      const tooltip = d3.select("body")
        .append("div")
        .attr("class", "tooltip")
        .style("position", "absolute")
        .style("background", "rgba(0,0,0,0.8)")
        .style("color", "#fff")
        .style("padding", "6px 12px")
        .style("border-radius", "5px")
        .style("pointer-events", "none")
        .style("font-size", "13px")
        .style("opacity", 0);
  
      // Draw bar for Country A
      svg.append("rect")
        .attr("x", x(0))
        .attr("y", y(this.selectedA))
        .attr("width", x(gdpA) - x(0))
        .attr("height", y.bandwidth())
        .attr("fill", "#e7b4aa")
        .on("mouseover", () => {
          tooltip.transition().duration(200).style("opacity", 0.9);
          tooltip.html(`<strong>${this.selectedA}</strong><br>GDP: $${gdpA.toLocaleString()}`);
        })
        .on("mousemove", (event) => {
          tooltip
            .style("left", (event.pageX + 15) + "px")
            .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", () => {
          tooltip.transition().duration(200).style("opacity", 0).remove();
        });
  
      // Draw bar for Country B
      svg.append("rect")
        .attr("x", x(0))
        .attr("y", y(this.selectedB))
        .attr("width", x(gdpB) - x(0))
        .attr("height", y.bandwidth())
        .attr("fill", "#953553")
        .on("mouseover", () => {
          tooltip.transition().duration(200).style("opacity", 0.9);
          tooltip.html(`<strong>${this.selectedB}</strong><br>GDP: $${gdpB.toLocaleString()}`);
        })
        .on("mousemove", (event) => {
          tooltip
            .style("left", (event.pageX + 15) + "px")
            .style("top", (event.pageY - 28) + "px");
        })
        .on("mouseout", () => {
          tooltip.transition().duration(200).style("opacity", 0).remove();
        });
  
      // Optional difference bar
      const diff = Math.abs(gdpA - gdpB);
      const lower = Math.min(gdpA, gdpB);
      svg.append("rect")
        .attr("x", x(lower))
        .attr("y", y(this.selectedB) + y.bandwidth() + 5)
        .attr("width", x(diff) - x(0))
        .attr("height", 10)
        .attr("fill", "#ccc");
  
      // Labels for each bar
      svg.append("text")
        .attr("x", x(gdpA) + 5)
        .attr("y", y(this.selectedA) + y.bandwidth() / 2 + 5)
        .text(gdpA.toLocaleString())
        .attr("font-size", "12px");
  
      svg.append("text")
        .attr("x", x(gdpB) + 5)
        .attr("y", y(this.selectedB) + y.bandwidth() / 2 + 5)
        .text(gdpB.toLocaleString())
        .attr("font-size", "12px");
  
      // Axes
      svg.append("g")
        .attr("transform", `translate(0, ${this.height - this.margin.bottom})`)
        .call(d3.axisBottom(x).ticks(5).tickFormat(d3.format(".2s")));
  
      svg.append("g")
        .attr("transform", `translate(${this.margin.left}, 0)`)
        .call(d3.axisLeft(y));
    }
  }
};
</script>
  
<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;  /* slightly darker blue for contrast */
  border: 1px ;
  padding: 10px;
  min-height: 635px; /* Adjust as needed for Difference Chart */
}

select {
  font-size: 14px;
}

.tooltip {
  transition: opacity 0.3s ease;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.3);
  z-index: 10;
}
</style>
