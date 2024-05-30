<template>
	<div class="container">
		<h1>研究样本说明</h1>
		<div id="map-container">
			<div class="text-container">
				<h2>高人口自然增长率地区</h2>
				<p>
					2000年-2023年期间，每一年的人口自然增长率均高于全国平均的地区&超过70%个自然年的人口自然增长率高于全国平均的地区（少数民族地区除外）
				</p>
				<h3>海南，青海，江西，云南，甘肃，贵州，广东，广西</h3>
				<h2>低人口自然增长率地区</h2>
				<p>
					2000年-2023年期间，每一年的人口自然增长率均低于全国平均的地区（少数民族地区除外）
				</p>
				<h3>天津，上海，辽宁，黑龙江，吉林，江苏</h3>
			</div>
			<div>
				<div class="controls">
					<label for="year">选择年份</label>
					<input
						v-model="selectedYear"
						type="number"
						min="2000"
						max="2023"
					/>
					<!-- <button @click="updateHeatmap">筛选</button> -->
				</div>
				<div ref="heatmap"></div>
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

	const width = 700;
	const height = 500;
	const colorScale = d3
		.scaleSequential(d3.interpolateYlOrRd)
		.domain([0, d3.max(data, (d) => d[year])]);

	// 使用D3地理图谱绘制中国地图
	const projection = d3
		.geoMercator()
		.center([105, 32])
		.scale(600)
		.translate([width / 2, height / 1.5]);

	const path = d3.geoPath().projection(projection);

	// 加载中国地理数据
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
					`人口自然增长率${provinceData[year]}`
				);
			})
			.on("mouseout", () => {
				const tooltip = d3.select("#tooltip-map");
				tooltip.style("display", "none");
			});
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
		.attr("width", 700)
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
</style>
