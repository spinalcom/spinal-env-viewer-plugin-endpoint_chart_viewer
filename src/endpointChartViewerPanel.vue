<!--
Copyright 2018 SpinalCom - www.spinalcom.com

This file is part of SpinalCore.

Please read all of the following terms and conditions
of the Free Software license Agreement ("Agreement")
carefully.

This Agreement is a legally binding contract between
the Licensee (as defined below) and SpinalCom that
sets forth the terms and conditions that govern your
use of the Program. By installing and/or using the
Program, you agree to abide by all the terms and
conditions stated or referenced herein.

If you do not agree to abide by these terms and
conditions, do not demonstrate your acceptance and do
not install or use the Program.
You should have received a copy of the license along
with this file. If not, see
<http://resources.spinalcom.com/licenses.pdf>.
-->

<template>
  <div class="endpoint-chart-viewer-panel">
    <div class="md-layout endpoint-chart-viewer-panel-topbtn-container">
      <md-button v-for="value in buttons"
                 :key="value"
                 class="md-layout-item topbtn"
                 :disabled="value === btnSelected"
                 :class="{'raise-disable': value === btnSelected}"
                 @click="onClick(value)">{{value}}</md-button>
      <md-button class="md-layout-item topbtn"
                 :disabled="value === btnSelected"
                 :class="{'raise-disable': 'CUSTOM' === btnSelected}"
                 @click="onClickCustom">CUSTOM</md-button>

    </div>
    <div class="md-layout md-alignment-center-center endpoint-chart-viewer-panel-chart-container">
      <plotlyCompoment :chartData="timeSeriesData"></plotlyCompoment>
    </div>
    <customDateIntervalDialog @closeDialog="closeDialogCustom"
                              :isOpen="isDialogCustomOpen"></customDateIntervalDialog>
  </div>
</template>
<script>
import plotlyCompoment from "./plotlyCompoment.vue";
import { ChartDataEndpoint } from "./ChartDataEndpoint.js";
import customDateIntervalDialog from "./customDateIntervalDialog.vue";
export default {
  name: "my_compo",
  components: {
    plotlyCompoment,
    customDateIntervalDialog
  },
  data() {
    return {
      isDialogCustomOpen: false,
      btnSelected: "1h",
      buttons: ["1h", "3h", "24h", "J-1", "3J", "7J"],
      timeSeriesData: []
    };
  },
  methods: {
    async toogleSelect(nodeId) {
      const index = this.timeSeriesData.findIndex(elem => {
        return elem.nodeId === nodeId;
      });
      if (index === -1) {
        let data;
        if (this.btnSelected === "CUSTOM") {
          data = new ChartDataEndpoint(
            nodeId,
            this.btnSelected,
            this.lastStart,
            this.lastEnd
          );
        } else {
          data = new ChartDataEndpoint(nodeId, this.btnSelected);
        }
        await data.init();
        this.timeSeriesData.push(data);
      } else {
        const endpointRemoved = this.timeSeriesData.splice(index, 1);
        for (let index = 0; index < endpointRemoved.length; index++) {
          endpointRemoved[index].uninit();
        }
      }
    },
    onClick(value) {
      this.btnSelected = value;
      for (let index = 0; index < this.timeSeriesData.length; index++) {
        this.timeSeriesData[index].changeInterval(value);
      }
    },
    onClickCustom() {
      this.isDialogCustomOpen = true;
    },
    closeDialogCustom(value) {
      const { start, end, valid } = value;
      this.isDialogCustomOpen = false;
      if (valid) {
        this.lastStart = start;
        this.lastEnd = end;
        this.btnSelected = "CUSTOM";
        for (let index = 0; index < this.timeSeriesData.length; index++) {
          this.timeSeriesData[index].changeCustomInterval(start, end);
        }
      }
    },
    opened(option) {
      return this.toogleSelect(option.selectedNode.id.get());
    },
    removed() {
      for (let index = 0; index < this.timeSeriesData.length; index++) {
        this.timeSeriesData[index].uninit();
      }
      this.timeSeriesData = [];
    },
    closed() {}
  }
};
</script>

<style>
.endpoint-chart-viewer-panel {
  height: 100%;
}
.endpoint-chart-viewer-panel,
.endpoint-chart-viewer-panel * {
  box-sizing: border-box;
}
.endpoint-chart-viewer-panel .endpoint-chart-viewer-panel-topbtn-container {
  border-bottom: 1px solid #a6a6a7;
  max-height: 36px;
}

.endpoint-chart-viewer-panel .topbtn {
  margin: unset;
  min-width: 3em;
}
.endpoint-chart-viewer-panel .endpoint-chart-viewer-panel-chart-container {
  height: calc(100% - 36px);
}
.raise-disable {
  color: #3a3a3a !important;
  background-color: #a6a6a7;
}
.endpoint-chart-viewer-panel .md-content {
  background-color: rgba(66, 66, 66,0.4);
}
</style>