<template>
	<div class="container">
		<h1>地域性差异</h1>
		<div id="map-container">
			<div class="text-container">
				<h2>高人口自然增长率地区</h2>
				<p>
					2000年-2023年期间，每一年的人口自然增长率均高于全国平均的地区&超过70%个自然年的人口自然增长率高于全国平均的地区
				</p>
				<h3>海南，青海，江西，云南，甘肃，贵州，广东，广西</h3>
				<h2>低人口自然增长率地区</h2>
				<p>
					2000年-2023年期间，每一年的人口自然增长率均低于全国平均的地区
				</p>
				<h3>天津，上海，辽宁，黑龙江，吉林，江苏</h3>
			</div>
			<div id="chart-controls-container">
				<div class="chart-container">
					<h2>人口自然增长率地图</h2>
					<div ref="heatmap"></div>
				</div>
				<div class="controls">
					<label for="year">选择年份</label>
					<input
						v-model="selectedYear"
						type="range"
						min="2000"
						max="2023"
                        class="year-slider"
					/>
					<span>{{ selectedYear }}</span>
				</div>
			</div>
		</div>
	</div>
	<div id="tooltip-map" class="tooltip">
		<p><span id="tooltip-province"></span></p>
		<p><span id="tooltip-rate"></span></p>
	</div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3";

const heatmap = ref(null);

const selectedYear = ref(2023);

let data = [];

async function loadData() {
	const csvData = await d3.csv(
		`${import.meta.env.BASE_URL}data/人口自然增长率数据.csv`
	);
	// console.log(csvData);
	data = csvData.map((d) => {
		const { 省份, ...years } = d;
		const formattedData = { province: 省份 };
		for (const [year, value] of Object.entries(years)) {
			formattedData[year] = +value;
		}
		return formattedData;
	});
	// console.log(data);
}

function drawHeatmap(year) {
	const svg = d3.select(heatmap.value).select("svg");
	svg.selectAll("*").remove();

	const width = 800;
	const height = 500;

	// 确定统一的比例尺范围
	const colorScale = d3
		.scaleSequential(d3.interpolateYlOrRd)
		.domain([-5, 10]);

	// 使用D3地理图谱绘制地图
	const projection = d3
		.geoMercator()
		.center([105, 32])
		.scale(600)
		.translate([width / 2, height / 1.5]);

	const path = d3.geoPath().projection(projection);

	// 加载地理数据
	d3.json(`${import.meta.env.BASE_URL}china.json`).then((geoData) => {
		svg.selectAll("path")
			.data(geoData.features)
			.join("path")
			.attr("d", path)
			.attr("fill", (d) => {
				const provinceData = data.find(
					(p) => p.province === d.properties.name
				);
				return provinceData ? colorScale(provinceData[year]) : "#ccc";
			})
			.attr("stroke", "#333")
			.attr("cursor", "pointer")
			.on("mouseover", (event, d) => {
				const tooltip = d3.select("#tooltip-map");
				tooltip
					.style("display", "block")
					.style("left", event.pageX + 10 + "px")
					.style("top", event.pageY + 10 + "px");
				const provinceData = data.find(
					(p) => p.province === d.properties.name
				);

				d3.select("#tooltip-province").text(`${d.properties.name}`);
				d3.select("#tooltip-rate").text(
					`人口自然增长率${provinceData[year]}‰`
				);
			})
			.on("mouseout", () => {
				const tooltip = d3.select("#tooltip-map");
				tooltip.style("display", "none");
			});

		// 添加图例
		const legendWidth = 20;
		const legendHeight = 300;
		const legendMargin = { top: 170, right: 50, bottom: 20, left: 20 };

		const legendSvg = svg
			.append("g")
			.attr("class", "legend")
			.attr(
				"transform",
				`translate(${width - legendMargin.right - legendWidth}, ${
					legendMargin.top
				})`
			);

		const gradient = legendSvg
			.append("defs")
			.append("linearGradient")
			.attr("id", "gradient")
			.attr("x1", "0%")
			.attr("y1", "100%")
			.attr("x2", "0%")
			.attr("y2", "0%");

		gradient
			.append("stop")
			.attr("offset", "0%")
			.attr("stop-color", d3.interpolateYlOrRd(0));

		gradient
			.append("stop")
			.attr("offset", "100%")
			.attr("stop-color", d3.interpolateYlOrRd(1));

		legendSvg
			.append("rect")
			.attr("x", 0)
			.attr("y", 0)
			.attr("width", legendWidth)
			.attr("height", legendHeight)
			.style("fill", "url(#gradient)");

		const legendScale = d3
			.scaleLinear()
			.domain([-5, 10]) // 统一的范围
			.range([legendHeight, 0]);

		const legendAxis = d3
			.axisRight(legendScale)
			.ticks(5)
			.tickFormat(d3.format(".1f"));

		legendSvg
			.append("g")
			.attr("transform", `translate(${legendWidth}, 0)`)
			.call(legendAxis)
			.selectAll("text")
			.attr("dy", "0.32em");
	});
}

function updateHeatmap() {
	drawHeatmap(selectedYear.value);
}

onMounted(async () => {
	await loadData();
	const svg = d3
		.select(heatmap.value)
		.append("svg")
		.attr("width", 800)
		.attr("height", 500);
	drawHeatmap(selectedYear.value);
});

watch(selectedYear, (newValue) => {
	drawHeatmap(newValue);
});
</script>

<style scoped>
#map-container {
	display: flex;
	justify-content: center;
	align-items: center;
	/* flex-direction: column; */
}
.tooltip {
	position: absolute;
	background-color: #f9f9f9;
	border: 1px solid #d3d3d3;
	padding: 5px;
	display: none;
}

#chart-controls-container{
    display: flex;
    flex-direction: column;
    align-items: center;

}
.controls {
	display: flex;
	align-items: center;
	gap: 10px;
}

.year-slider {
	-webkit-appearance: none;
	width: 300px; /* 调整滑动条的宽度 */
	background: transparent;
}

.year-slider:focus {
	outline: none;
}

.year-slider::-webkit-slider-runnable-track {
	width: 100%;
	height: 8px;
	background: #fec763; /* 淡黄色 */
	border-radius: 4px;
	cursor: pointer;
}

.year-slider::-webkit-slider-thumb {
	-webkit-appearance: none;
	width: 20px;
	height: 20px;
	background: #f33e26;
	border-radius: 50%;
	cursor: pointer;
	margin-top: -6px; /* 调整滑块位置 */
}

.year-slider::-moz-range-track {
	width: 100%;
	height: 8px;
	background: #fec763; /* 淡黄色 */
	border-radius: 4px;
	cursor: pointer;
}

.year-slider::-moz-range-thumb {
	width: 20px;
	height: 20px;
	background: #f33e26;
	border-radius: 50%;
	cursor: pointer;
}

</style>
