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
  <md-dialog class="endpoint-chart-viewer-panel-dialog-custom-interval"
             :md-active.sync="isOpenComputed"
             :md-close-on-esc=true
             :md-click-outside-to-close=true
             :md-closed=onClose>
    <md-dialog-content class="md-scrollbar">
      <div class="md-layout">
        <div class="md-layout-item endpoint-chart-viewer-panel-dialog-custom-interval-calendar-container">
          <h2>Start : {{start}}</h2>
          <VueCtkDateTimePicker :noHeader=true
                                :max-date="end"
                                :dark=true
                                :hint="hint"
                                :format="format"
                                :noButtonNow=false
                                v-model="start"
                                :inline=true>
          </VueCtkDateTimePicker>
        </div>
        <div class="md-layout-item endpoint-chart-viewer-panel-dialog-custom-interval-calendar-container">
          <h2>End : {{end}}</h2>
          <VueCtkDateTimePicker :noHeader=true
                                :max-date="today"
                                :min-date="endStart"
                                :dark=true
                                :hint="hint"
                                :format="format"
                                :noButtonNow=false
                                v-model="end"
                                :inline=true>
          </VueCtkDateTimePicker>
        </div>
      </div>
    </md-dialog-content>
    <md-dialog-actions>
      <md-button class="md-primary"
                 @click="closeDialog(false)">Close</md-button>
      <md-button class="md-primary"
                 @click="closeDialog(true)">Valid</md-button>
    </md-dialog-actions>
  </md-dialog>
</template>

<script>
// import VueCtkDateTimePicker from "vue-ctk-date-time-picker";

export default {
  name: "customDateIntervalDialog",
  props: ["isOpen"],
  data() {
    const today = new Date();
    today.setHours(0, 0, 0, 0);
    return {
      hint: "Select the starting point to show",
      format: "YYYY-MM-DD HH:mm",
      today: today.toISOString(),
      start: null,
      end: null,
      valid: false
    };
  },
  computed: {
    isOpenComputed: {
      get() {
        return this.isOpen;
      },
      set(value) {
        if (value === false) {
          this.onClose();
        }
      }
    },
    endStart() {
      if (this.start === null) return null;
      const startDate = new Date(this.start);
      startDate.setHours(0, 0, 0, 0);
      return startDate.toISOString();
    }
  },
  methods: {
    closeDialog(valid) {
      this.valid = valid;
      this.isOpenComputed = false;
    },
    onClose() {
      let valid = this.valid;
      if (this.end === null || this.start === null) {
        valid = false;
      }
      this.valid = false;
      this.$emit("closeDialog", {
        start: this.start,
        end: this.end,
        valid: valid
      });
    }
  }
};
</script>

<style>
.endpoint-chart-viewer-panel-dialog-custom-interval {
  width: 80vw;
}
.endpoint-chart-viewer-panel-dialog-custom-interval > .md-dialog-container {
  max-height: 80vh;
}
.endpoint-chart-viewer-panel-dialog-custom-interval
  > .md-dialog-container
  > .md-dialog-content.md-scrollbar::-webkit-scrollbar {
  background-color: #121212;
}
.endpoint-chart-viewer-panel-dialog-custom-interval
  > .md-dialog-container
  > .md-dialog-content.md-scrollbar::-webkit-scrollbar-thumb {
  background-color: #737374;
}

.endpoint-chart-viewer-panel-dialog-custom-interval-calendar-container {
  border: 5px double #cbcbcb;
  margin: 20px;
}
.endpoint-chart-viewer-panel-dialog-custom-interval-calendar-container > h2 {
  text-align: center;
}
</style>
