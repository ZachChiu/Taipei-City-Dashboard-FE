<!-- Developed by Taipei Urban Intelligence Center 2023 -->

<script setup>
import { ref, computed, onMounted, nextTick } from "vue";
import { useMapStore } from "../../store/mapStore";

const props = defineProps([
	"chart_config",
	"activeChart",
	"series",
	"map_config",
]);
const mapStore = useMapStore();

const chartOptions = ref({
	chart: {
		type: "scatter",
		toolbar: {
			show: false,
		},
		zoom: {
			enabled: true,
			type: "xy",
		},
	},
	grid: {
		show: false,
	},
	tooltip: {
		custom: function ({ series, seriesIndex, dataPointIndex, w }) {
			return (
				'<div class="chart-tooltip">' +
				"<h6>" +
				props.series[seriesIndex].data[dataPointIndex].name +
				"</h6>" +
				"<span>" +
				`${props.chart_config.xaxisUnit}:${series[seriesIndex][dataPointIndex]}` +
				"<br />" +
				`${props.chart_config.yaxisUnit}:${w.globals.seriesX[seriesIndex][dataPointIndex]}` +
				"</span>" +
				"</div>"
			);
		},
		followCursor: true,
	},
	xaxis: {
		tickAmount: 5,
		// min: 0,
		// max: 0,
		axisTicks: {
			show: false,
		},
		tooltip: {
			enabled: false,
		},
	},
	yaxis: {
		tickAmount: 5,

		// min: 0,
		// max: 0,
	},
	stroke: {
		colors: ["#282a2c"],
		show: true,
		width: 0,
	},
});

const xMax = ref(0);
const yMax = ref(0);

const findMaxMin = () => {
	if (props.series.length) {
		const seriesData = props.series;
		for (let i = 0; i < seriesData.length; i++) {
			const serie = seriesData[i];
			for (let j = 0; j < serie.data.length; j++) {
				const position = serie.data[j];
				xMax.value = Math.max(Math.abs(position[0]), xMax.value);
				yMax.value = Math.max(Math.abs(position[0]), yMax.value);
			}
		}
	}
};

onMounted(() => {
	// findMaxMin();
	// chartOptions.value.xaxis.min = -xMax.value;
	// chartOptions.value.xaxis.max = xMax.value;
	// chartOptions.value.yaxis.min = -yMax.value;
	// chartOptions.value.yaxis.max = yMax.value;
	nextTick(() => {
		drawBg();
	});
});

const drawBg = () => {
	if (props.chart_config.backgrounds.length === 0) {
		return;
	}
	const bgs = props.chart_config.backgrounds;
	const { width, height } = getSize();
	const ns = "http://www.w3.org/2000/svg";
	const foreignObject = document.createElementNS(ns, "foreignObject");

	foreignObject.setAttribute("x", 0);
	foreignObject.setAttribute("y", 0);
	foreignObject.setAttribute("width", width);
	foreignObject.setAttribute("height", height);

	for (let i = 0; i < bgs.length; i++) {
		const bg = bgs[i];
		const div = document.createElement("div");
		div.setAttribute(
			"style",
			`position: absolute; background: ${bg.color}; width: ${bg.width}; height: ${bg.height}; top: ${bg.top}; left: ${bg.left};`
		);
		foreignObject.appendChild(div);
	}
	document.querySelector(".apexcharts-grid").append(foreignObject);
};

const getSize = () => {
	const domData = document
		.querySelector(".apexcharts-grid")
		.getBoundingClientRect();
	return {
		width: domData.width,
		height: domData.height,
	};
};

const onChartUpdated = () => {
	drawBg();
};

const processedSeries = computed(() => {
	return props.series.map((perSeries) => {
		return {
			name: perSeries.name,
			data: perSeries.data.map((obj) => obj.coordinates),
		};
	});
});
</script>

<template>
	<div v-if="activeChart === 'ScatterChart'" class="scatter-chart">
		<apexchart
			width="100%"
			height="250px"
			type="scatter"
			:options="chartOptions"
			:series="processedSeries"
			@updated="onChartUpdated"
		></apexchart>
	</div>
</template>
