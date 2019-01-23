
<template>
  <md-content class="chart-data-endpoint-graph-container">
    <md-content id="chart-data-endpoint-graph-container-plotgraph"></md-content>
    <md-empty-state v-if="isReady && chartData.length <= 0"
                    class="chart-data-endpoint-graph-container-empty"
                    md-icon="show_chart"
                    md-label="No Endpoint selected">
    </md-empty-state>
    <md-empty-state v-else-if="isReady && !haveData"
                    class="chart-data-endpoint-graph-container-empty"
                    md-icon="show_chart"
                    md-label="No Data"
                    md-description="No data to show in the Endpoint(s) Selected !">
    </md-empty-state>
    <md-button class="md-icon-button md-primary md-raised chart-data-endpoint-graph-container-option-button"
               @click="optionOpen = true">
      <md-icon>settings</md-icon>
    </md-button>
    <chartOptionDialog :isOpen="optionOpen"
                       :layoutOption="this.layout"
                       @closeDialog="optionOpen = false"
                       @updateOptions="updateOptions"></chartOptionDialog>
  </md-content>
</template>

<script>
import chartOptionDialog from "./chartOptionDialog.vue";
import { clearInterval } from "timers";
const globalType = typeof window === "undefined" ? global : window;
export default {
  name: "plotlyComponent",
  components: { chartOptionDialog },
  data() {
    return {
      optionOpen: false,
      isReady: false,
      haveData: false,
      layout: {
        margin: {
          b: 90,
          t: 8,
          pad: 4
        },
        font: {
          size: 15,
          color: "#FFFFFF"
        },
        paper_bgcolor: "rgba(0,0,0,0)",
        plot_bgcolor: "rgba(0,0,0,0)",
        xaxis: {
          rangeslider: null,
          showgrid: false,
          showline: true,
          type: "date",
          tickformat: "%d/%m/%Y %H:%M"
        },
        yaxis: {
          showgrid: false,
          showline: true
        },
        showlegend: true,
        legend: {
          orientation: "v",
          x: null,
          y: null,
          traceorder: "normal",
          font: {
            family: "sans-serif",
            size: 12,
            color: "#FFF"
          },
          bgcolor: "rgba(34,34,34,.9)",
          bordercolor: "#424242",
          borderwidth: 2
        }
      }
    };
  },
  props: ["chartData"],
  mounted() {
    this.init();
    this._graph_ = this.createGraph();

    this.intervalResize = setInterval(() => {
      this.resize(this._graph_.gd, this._graph_.gd3);
    }, 1000);
  },
  watch: {
    chartData: {
      deep: true,
      handler: function(val) {
        this.updateGraph(val);
      }
    }
  },
  beforeDestroy() {
    clearInterval(this.intervalResize);
  },
  methods: {
    init() {
      this.promUpdate = null;
      this.rect = {
        w: "100px",
        h: "100px"
      };
      this.my_chartData = [];
      this._graph_ = null;
    },
    updateObj(obj, data) {
      for (const key in data) {
        if (data.hasOwnProperty(key)) {
          if (
            Array.isArray(data[key]) === false &&
            typeof data[key] === "object" &&
            data[key] !== null
          ) {
            if (typeof obj[key] === "undefined" || obj[key] === null) {
              obj[key] = {};
            }
            this.updateObj(obj[key], data[key]);
          } else {
            obj[key] = data[key];
          }
        }
      }
    },
    updateOptions(data) {
      this.updateObj(this.layout, data);
      globalType.Plotly.react(this._graph_.gd, this.my_chartData, this.layout, {
        modeBarButtonsToRemove: ["sendDataToCloud"],
        displaylogo: false,
        responsive: true
      });
    },
    updateGraph: function(chartData) {
      for (let index = 0; index < chartData.length; index++) {
        const element = chartData[index];
        let found = false;
        for (let idx = 0; idx < this.my_chartData.length; idx++) {
          const myElement = this.my_chartData[idx];
          if (element.nodeId === myElement.nodeId) {
            if (
              element.x.length !== myElement.x.length ||
              (element.x[0] && element.x[0] !== myElement.x[0])
            ) {
              (myElement.x = element.x), (myElement.y = element.y);
            }
            found = true;
            break;
          }
        }
        if (found === false) {
          this.my_chartData.push({
            nodeId: element.nodeId,
            mode: "lines",
            type: "scatter",
            name: element.name,
            x: element.x,
            y: element.y
          });
        }
      }
      const toDelete = [];
      for (let idx = 0; idx < this.my_chartData.length; idx++) {
        const myElement = this.my_chartData[idx];
        let found = false;
        for (let index = 0; index < chartData.length; index++) {
          const element = chartData[index];
          if (element.nodeId === myElement.nodeId) {
            found = true;
          }
        }
        if (found === false) {
          toDelete.push(myElement);
        }
      }

      for (let index = 0; index < toDelete.length; index++) {
        const element = toDelete[index];
        const idx = this.my_chartData.indexOf(element);
        this.my_chartData.splice(idx, 1);
      }
      globalType.Plotly.react(this._graph_.gd, this.my_chartData, this.layout, {
        modeBarButtonsToRemove: ["sendDataToCloud"],
        displaylogo: false,
        responsive: true
      });
      for (let index = 0; index < this.my_chartData.length; index++) {
        const element = this.my_chartData[index];
        if (element.x.length > 0) {
          this.haveData = true;
          this.isReady = true;
          return;
        }
      }
      this.haveData = false;
      this.isReady = true;
    },

    createGraph: function() {
      var d3 = globalType.Plotly.d3;
      var graphWidth = 100,
        graphHeight = 100;
      var gd3 = d3
        .select(
          document.getElementById(
            "chart-data-endpoint-graph-container-plotgraph"
          )
        )
        .style({
          width: graphWidth + "%",
          height: graphHeight + "%"
        });
      var gd = gd3.node();
      return { gd: gd, gd3: gd3 };
    },
    resize: function(gd, gd3) {
      let w = gd3.style("width");
      let h = gd3.style("height");
      if (this.rect.w !== w || this.rect.h !== h) {
        this.rect.w = w;
        this.rect.h = h;
        globalType.Plotly.Plots.resize(gd);
      }
    }
  }
};
</script>
<style>
.chart-data-endpoint-graph-container-plotgraph
  .js-plotly-plot
  .plotly
  .modebar {
  left: 0;
}
</style>

<style scoped>
.chart-data-endpoint-graph-container {
  height: 100%;
  width: 100%;
}
.chart-data-endpoint-graph-container-empty {
  top: 0;
  position: absolute;
  width: 100%;
  height: 100%;
  max-width: unset;
}
.chart-data-endpoint-graph-container-option-button {
  bottom: 5px;
  left: 0;
  position: absolute;
}
</style>
