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
			<md-button
				v-for="value in buttons"
				:key="value"
				class="md-layout-item topbtn"
				:disabled="value === btnSelected"
				:class="{ 'raise-disable': value === btnSelected }"
				@click="onClick(value)"
				>{{ value }}</md-button
			>
			<md-button
				class="md-layout-item topbtn"
				:disabled="value === btnSelected"
				:class="{ 'raise-disable': 'CUSTOM' === btnSelected }"
				@click="onClickCustom"
				>CUSTOM</md-button
			>
		</div>
		<div
			class="
				md-layout md-alignment-center-center
				endpoint-chart-viewer-panel-chart-container
			"
			:class="{ autodeskv6: isviewerV6 }"
		>
			<plotlyCompoment
				:data="timeSeriesData"
				:reloadData="reloadData"
			></plotlyCompoment>
		</div>
		<customDateIntervalDialog
			@closeDialog="closeDialogCustom"
			:dateAvailable="dateAvailable"
			:isOpen="isDialogCustomOpen"
		></customDateIntervalDialog>
	</div>
</template>
<script>
	import plotlyCompoment from "./plotlyCompoment.vue";
	import { ChartDataEndpoint } from "./ChartDataEndpoint.js";
	import customDateIntervalDialog from "./customDateIntervalDialog.vue";
	import union from "lodash.union";
	import { timeSeriesMap, resetTimeSeriesMap } from "./map";

	export default {
		name: "my_compo",
		components: {
			plotlyCompoment,
			customDateIntervalDialog,
		},
		data() {
			return {
				reloadData: 0,
				isDialogCustomOpen: false,
				btnSelected: "1h",
				buttons: ["1h", "3h", "24h", "J-1", "3J", "7J"],
				timeSeriesData: [],
				dateAvailable: [],
			};
		},
		computed: {
			isviewerV6() {
				return parseInt(window.LMV_VIEWER_VERSION) === 6;
			},
		},
		methods: {
			async toogleSelect(nodeId) {
				const index = this.timeSeriesData.findIndex((elem) => {
					return elem === nodeId;
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
					timeSeriesMap.set(nodeId, data);
					this.timeSeriesData.push(nodeId);
				} else {
					const idToRemove = this.timeSeriesData[index];

					// for (let index of endpointToRemove) {
					// 	const t = timeSeriesMap.get(index);
					// 	t.uninit();
					// 	timeSeriesMap.delete(index);
					// }

					// const promises = endpointToRemove.map((index) => {
					const t = timeSeriesMap.get(idToRemove);
					timeSeriesMap.delete(idToRemove);
					await t.uninit();
					// });

					// await Promise.all(promises);

					this.timeSeriesData.splice(index, 1);
				}
				let tmp = this.timeSeriesData.map((nodeId) => {
					const element = timeSeriesMap.get(nodeId);
					return element.dateAvailable;
				});
				tmp = union(...tmp);
				tmp.sort((a, b) => a - b);
				this.dateAvailable = tmp;
			},
			async onClick(value) {
				this.btnSelected = value;
				const promises = this.timeSeriesData.map((index) => {
					const t = timeSeriesMap.get(index);
					return t.changeInterval(value);
				});
				// for (let index of this.timeSeriesData) {
				// 	const t = timeSeriesMap.get(index);
				// 	await t.changeInterval(value);
				// }
				await Promise.all(promises);
				this.reloadData++;
			},
			onClickCustom() {
				this.isDialogCustomOpen = true;
			},
			async closeDialogCustom(value) {
				const { start, end, valid } = value;
				this.isDialogCustomOpen = false;
				if (valid) {
					this.lastStart = start;
					this.lastEnd = end;
					this.btnSelected = "CUSTOM";
					// for (let index of this.timeSeriesData) {
					// 	const t = timeSeriesMap.get(index);
					// 	await t.changeCustomInterval(start, end);
					// }
					const promises = this.timeSeriesData.map((index) => {
						const t = timeSeriesMap.get(index);
						return t.changeCustomInterval(start, end);
					});
					await Promise.all(promises);
					this.reloadData++;
				}
			},
			opened(option) {
				return this.toogleSelect(option.selectedNode.id.get());
			},
			removed() {
				for (let index of this.timeSeriesData) {
					const t = timeSeriesMap.get(index);
					t.uninit();
				}
				this.timeSeriesData = [];
				resetTimeSeriesMap();
			},
			closed() {},
		},
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
	.endpoint-chart-viewer-panel
		.endpoint-chart-viewer-panel-chart-container.autodeskv6 {
		height: calc(100% - 56px);
	}

	.raise-disable {
		color: #3a3a3a !important;
		background-color: #a6a6a7;
	}
	.endpoint-chart-viewer-panel .md-content {
		background-color: rgba(66, 66, 66, 0.4);
	}
</style>
