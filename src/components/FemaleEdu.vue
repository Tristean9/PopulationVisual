<template>
	<h3>6岁及以上大专以上学历女性占比</h3>
	<div ref="chart"></div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const width = 800;
const height = 400;
const margin = { top: 20, right: 30, bottom: 50, left: 60 };
const chart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(`${import.meta.env.BASE_URL}data/受教育程度.csv`);
	// console.log(data);

	const chartData = data
		.map((d) => ({
			year: d["年份"],
			value: +d["大专及以上女性人口占比"],
		}))
		.reverse();

	// console.log(chartData);

	return chartData;
}

// 绘制折线图
function drawChart(svg, data) {
	const xScale = d3
		.scaleBand()
		.domain(data.map((d) => d.year))
		.range([0, width - margin.left - margin.right])
		.padding(0.1);

	const yScale = d3
		.scaleLinear()
		.domain([
			d3.min(data, (d) => d.value) * 0.95,
			d3.max(data, (d) => d.value) * 1.05,
		])
		.range([height - margin.top - margin.bottom, 0]);

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left},${margin.top})`);

	const line = d3
		.line()
		.x((d) => xScale(d.year) + xScale.bandwidth() / 2)
		.y((d) => yScale(d.value));

	g.append("path")
		.datum(data)
		.attr("fill", "none")
		.attr("stroke", "steelblue")
		.attr("stroke-width", 2)
		.attr("d", line);

	// 添加X轴
	g.append("g")
		.attr("class", "x-axis")
		.attr(
			"transform",
			`translate(0, ${height - margin.top - margin.bottom})`
		)
		.call(d3.axisBottom(xScale))
		.append("text")
		.attr("x", (width - margin.left - margin.right) / 2)
		.attr("y", 40)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.text("年份");

	// 添加Y轴
	g.append("g").attr("class", "y-axis").call(d3.axisLeft(yScale))
		.append("text")
		.attr("transform", "rotate(-90)")
		.attr("x", -(height - margin.top - margin.bottom) / 2)
		.attr("y", -50)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.text("大专及以上女性人口占比 (%)");
}

onMounted(async () => {
	const data = await loadData();

	const svg = d3
		.select(chart.value)
		.append("svg")
		.attr("width", width)
		.attr("height", height);

	drawChart(svg, data);
});
</script>
<style scoped>
.axis path,
.axis line {
	fill: none;
	stroke: black;
	shape-rendering: crispEdges;
}

.axis text {
	font-family: sans-serif;
	font-size: 12px;
}
</style>
