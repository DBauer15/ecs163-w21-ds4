<!-- This file will be used to layout and connect your views -->
<template>
  <div id="app">
    <!-- This is only a basic setup to display your views. Use HTML and CSS to create your layout using these basic commands -->
    <ScatterPlot
      class="scatter-plot"
      v-if="dataset"
      chartId="poke1"
      title="Pokemon Overview"
      :dataset="dataset"
      attribX="relStrength"
      labelX="Relative Strength"
      attribY="relCatchRate"
      labelY="Relative Catch Rate"
      attribColor="Type_1"
      attribGlyph="isLegendary"
      attribGlyphValNormal="False"
      attribGlyphValSpecial="True"
      @item-selected="pokemonSelected"
      :width="800"
      :height="600"
    />
    <BarChart 
      class="bar-chart" 
      v-if="datasetSelected" 
      chartId="poke2" 
      :title="`Details about ${datasetSelected.name}`" 
      :dataset="datasetSelected.entries" 
      attribX="name" 
      attribY="value"  
      labelY="Stat Strength"
      :width="800"
      :height="600" />
  </div>
</template>

<script>
import * as d3 from "d3";

// Try to use descriptive names like 'OverviewScatterPlot' or 'DetailLineChart'
import ScatterPlot from "./components/ScatterPlot.vue";
import BarChart from "./components/BarChart.vue";

export default {
  name: "App",
  components: {
    ScatterPlot,
    BarChart,
  },
  data() {
    return {
      dataset: null,
      datasetSelected: null,
    };
  },
  created() {
    d3.csv("/data/pokemon.csv", d3.autoType).then((data) => {
      const maxTotal = d3.max(data, (d) => d.Total);
      const maxCatchRate = d3.max(data, (d) => d.Catch_Rate);

      this.dataset = d3.map(data, (d) => {
        return Object.assign(d, {
          relStrength: d.Total / maxTotal,
          relCatchRate: d.Catch_Rate / maxCatchRate,
        })
      })

      this.pokemonSelected(this.dataset[0])
    });
  },
  methods: {
    pokemonSelected(p) {
      let pokemon = d3.filter(this.dataset, d => d.Number == p.Number)[0]
      
      let entries = []
      for (const stat of ['HP', 'Attack', 'Defense', 'Sp_Atk', 'Sp_Def', 'Speed']) {
        entries.push({
          name: stat,
          value: pokemon[stat]
        })
      }

      this.datasetSelected = {
        name: pokemon.Name,
        entries: entries
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.scatter-plot {
  width: 800px;
  float: left;
}

.bar-chart {
  width: 800px;
  float: left;
}
</style>
