<template>
  <div>
    <h1 class="dashboard-title">Countries Comparison Dashboard</h1>
    <div class="container">
      <!-- Your dashboard layout code here -->
    </div> </div>
  <div class="container">
    <!-- First row: Two charts -->
    <div class="row">
      <div class="col-md-6 mb-4">
        <h5>Stacked Bar Chart</h5>
        <stacked-bar-chart></stacked-bar-chart>
      </div>
      <div class="col-md-6 mb-4">
        <h5>Donut Chart</h5>
        <donut-chart></donut-chart>
      </div>
    </div>

    <!-- Second row: Two charts -->
    <div class="row">
      <div class="col-md-6 mb-4">
        <h5>Parallel Coordinates Chart</h5>
        <parallel-coordinates-chart></parallel-coordinates-chart>
      </div>
      <div class="col-md-6 mb-4">
        <h5>Force Directed Graph- Countries with same official language </h5>
        <force-directed-graph></force-directed-graph>
      </div>
    </div>

    <!-- Third row: Two charts -->
    <div class="row">
      <div class="col-md-6 mb-4">
        <h5>GDP Difference Chart</h5>
        <difference-chart></difference-chart>
      </div>
      <div class="col-md-6 mb-4">
        <h5>CirclePackingChart</h5>
        <circle-packing-chart></circle-packing-chart>
      </div>
    </div>
  </div>
  <footer class="dashboard-footer">
    <p>
      Dashboard generated with the help of 
      <a href="https://chatgpt.com/share/67e72390-0028-8008-b10b-2abfea367eed" target="_blank" rel="noopener noreferrer">ChatGPT1</a>.
      <a href="https://chatgpt.com/share/67e723c0-e900-8008-8239-bd01dbc35c79" target="_blank" rel="noopener noreferrer">ChatGPT2</a>.
      <a href="https://chatgpt.com/share/67e723dc-4094-8008-98aa-ada0a1c2bc64" target="_blank" rel="noopener noreferrer">ChatGPT3</a>.
    </p>
    <p>I used ChatGPT's help for:</p>
    <ul>
      <li>
        Designing and structuring my dashboard layout: Creating a clear grid system with Bootstrap and custom placeholder styling to ensure each chart has its own section.
      </li>
      <li>
        Debugging and enhancing D3 visualizations: Improving the functionality and interactivity of charts (stacked bar, donut, parallel coordinates, force-directed graph, and circle packing) including tooltips and zoom features.
      </li>
      <li>
        Customizing color schemes: Adjusting default palettes to use my specified colours (like "#953553" and "#e7b4aa") and exploring ways to create custom shades.
      </li>
      <li>
        Refining UI components: Modifying elements such as dropdown widths, repositioning legends, and ensuring tooltips display accurate data by matching CSV column names correctly.
      </li>
      <li>
        Enhancing dashboard presentation: Adding a centered title and a footer that credits ChatGPT's assistance, which improves the overall look and transparency of the dashboard.
      </li>
    </ul>

    </footer>
  
</template>

<script>
import StackedBarChart from './components/StackedBarChart.vue';
import DonutChart from './components/DonutChart.vue';
import ParallelCoordinatesChart from './components/ParallelCoordinatesChart.vue';
import ForceDirectedGraph from './components/ForceDirectedGraph.vue';
import DifferenceChart from './components/DifferenceChart.vue';
import CirclePackingChart from './components/Circle.vue';

export default {
  name: 'App',
  components: {
    StackedBarChart,
    DonutChart,
    ParallelCoordinatesChart,
    ForceDirectedGraph,
    DifferenceChart,
    CirclePackingChart
  }
};
</script>

<style>
html, body {
  background-color: #b4a19b!important; /* Change to your desired color */
  min-height: 100%;
  margin: 0;
  padding: 0;
}
.dashboard-title {
  text-align: center;
  margin-top: 20px;
  font-size: 2rem;
  font-weight: bold;
}
</style>
