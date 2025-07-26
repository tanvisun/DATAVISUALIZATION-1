<template>
  <div class="chart-placeholder">
    <div ref="chart"></div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  name: "ForceDirectedGraph ",
  data() {
    return {
      nodes: [],
      links: [],
      width: 600,
      height: 500,
      svg: null,
    };
  },
  mounted() {
    this.loadData();
  },
  methods: {
    loadData() {
      d3.csv('/DATAVISUALIZATION-1/data/worlddata.csv').then(data => {
        // Filter only countries where official language is Arabic.
        let filteredData = data.filter(d => {
          return d["Official language"] &&
                 d["Official language"].trim().toLowerCase() === "arabic";
        });
  
        // Create nodes.
        const nodes = filteredData.map(d => ({
          id: d.Country,
          country: d.Country,
          officialLanguage: d["Official language"]
        }));
  
        // Link every pair since they share the same language.
        const links = [];
        for (let i = 0; i < nodes.length; i++) {
          for (let j = i + 1; j < nodes.length; j++) {
            links.push({
              source: nodes[i].id,
              target: nodes[j].id
            });
          }
        }
  
        this.nodes = nodes;
        this.links = links;
        this.createChart();
      }).catch(error => console.error("Error loading CSV data:", error));
    },
    createChart() {
      // Clear any previous chart elements.
      d3.select(this.$refs.chart).selectAll("*").remove();
  
      this.svg = d3.select(this.$refs.chart)
        .append("svg")
        .attr("width", this.width)
        .attr("height", this.height);
  
      const color = d3.scaleOrdinal()
        .domain(this.nodes.map(d => d.id))
        .range(d3.schemeReds[9].slice(4));
  
      const simulation = d3.forceSimulation(this.nodes)
        .force("link", d3.forceLink(this.links).id(d => d.id).distance(150))
        .force("charge", d3.forceManyBody().strength(-200))
        .force("center", d3.forceCenter(this.width / 2, this.height / 2));
  
      const link = this.svg.append("g")
        .attr("class", "links")
        .selectAll("line")
        .data(this.links)
        .enter().append("line")
        .attr("stroke", "#aaa")
        .attr("stroke-width", 1);
  
      const node = this.svg.append("g")
        .attr("class", "nodes")
        .selectAll("circle")
        .data(this.nodes)
        .enter().append("circle")
        .attr("r", 10)
        .attr("fill", d => color(d.id))
        .call(d3.drag()
          .on("start", dragstarted)
          .on("drag", dragged)
          .on("end", dragended));
  
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
  
      node.on("mouseover", function(event, d) {
          tooltip.transition().duration(200).style("opacity", 0.9);
          tooltip.html(
            `<strong>${d.country}</strong><br/>Language: ${d.officialLanguage}`
          )
          .style("left", (event.pageX + 10) + "px")
          .style("top", (event.pageY - 28) + "px");
      })
      .on("mousemove", function(event) {
          tooltip.style("left", (event.pageX + 10) + "px")
                 .style("top", (event.pageY - 28) + "px");
      })
      .on("mouseout", function() {
          tooltip.transition().duration(200).style("opacity", 0);
      });
  
      simulation.on("tick", () => {
        link.attr("x1", d => d.source.x)
            .attr("y1", d => d.source.y)
            .attr("x2", d => d.target.x)
            .attr("y2", d => d.target.y);
        node.attr("cx", d => d.x)
            .attr("cy", d => d.y);
      });
  
      function dragstarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }
      function dragged(event, d) {
        d.fx = event.x;
        d.fy = event.y;
      }
      function dragended(event, d) {
        if (!event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }
    }
  }
};
</script>
  
<style scoped>
.chart-placeholder {
  background-color: #e4d9d5bb;
  border: 1px dashed #ccc;
  padding: 10px;
  min-height: 520px;
}
.tooltip {
  font-family: Arial, sans-serif;
  font-size: 12px;
}
</style>
