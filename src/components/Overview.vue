<template>
	<h1>中国人口数据总览</h1>
	<div ref="lineChart"></div>
	<div ref="barChart"></div>
	<p>
		中国人口自然增长率在2000年后整体呈现逐渐降低趋势，在22年和23年出现负增长。22年的自然增长率为-0.6%，23年为-1.4%
	</p>
	<div id="tooltip">
		<p><span id="tooltip-year"></span></p>
		<p><span id="tooltip-population"></span></p>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const barChart = ref(null);
const lineChart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/人口增长情况（2000-2023）.csv`
	);

	// console.log(data);

	const lineData = data.map((d) => ({
		year: d["指标"],
		population: +d["年末总人口(万人)"],
	}));

	const barData = data.map((d) => ({
		year: d["指标"],
		growthRate: +d["增长率"],
	}));

	return { lineData, barData };
}

// 绘制折线图
function drawLineChart(svg, data) {
	const margin = { top: 20, right: 30, bottom: 30, left: 50 };
	const width = 800 - margin.left - margin.right;
	const height = 500 - margin.top - margin.bottom;

	const xScale = d3
		.scaleBand()
		.domain(data.map((d) => d.year))
		.range([0, width])
		.padding(0.1);

	const yScale = d3
		.scaleLinear()
		.domain([
			d3.min(data, (d) => d.population) * 0.95,
			d3.max(data, (d) => d.population) * 1.05,
		])
		.range([height, 0]);

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left}, ${margin.top})`);

	const line = d3
		.line()
		.x((d) => xScale(d.year) + xScale.bandwidth() / 2)
		.y((d) => yScale(d.population));

	g.append("path")
		.datum(data)
		.attr("fill", "none")
		.attr("stroke", "steelblue")
		.attr("stroke-width", 2)
		.attr("d", line);

	// 添加圆点
	g.selectAll("circle")
		.data(data)
		.join("circle")
		.attr("cx", (d) => xScale(d.year) + xScale.bandwidth() / 2)
		.attr("cy", (d) => yScale(d.population))
		.attr("r", 4)
		.attr("fill", "steelblue")
		.attr("cursor", "pointer")
		.on("mouseover", (event, d) => {
			const tooltip = d3.select("#tooltip");
			tooltip
				.style("display", "block")
				.style("left", event.pageX + 10 + "px")
				.style("top", event.pageY + 10 + "px");

			d3.select("#tooltip-year").text(`年份：${d.year}`);
			d3.select("#tooltip-population").text(
				`总人口：${d.population}万人`
			);
		})
		.on("mouseout", () => {
			const tooltip = d3.select("#tooltip");
			tooltip.style("display", "none");
		});

	// 添加X轴
	g.append("g")
		.attr("transform", `translate(0, ${height})`)
		.call(d3.axisBottom(xScale));

	// 添加Y轴
	g.append("g").call(d3.axisLeft(yScale));

	// 添加竖直虚线
	const xPosition = xScale("2020");
	g.append("line")
		.attr("x1", xPosition + 10)
		.attr("y1", 0)
		.attr("x2", xPosition + 10)
		.attr("y2", height)
		.attr("stroke", "gray")
		.attr("stroke-dasharray", "4");

	// 添加文本
	g.append("text")
		.attr("x", xPosition + 10)
		.attr("y", 200)
		.attr("font-size", 12)
		.text("新冠首个确证病例发病");

	g.append("text")
		.attr("x", xPosition + 10)
		.attr("y", 220)
		.attr("font-size", 12)
		.text("新冠首个确证病例发病")
		.text("（2019.12.1）");

	g.append("text")
		.attr("x", 10)
		.attr("y", 0)
		.attr("font-size", 12)
		.text("年末总人口（万人）");

	g.append("text")
		.attr("x", width - 4)
		.attr("y", height - 10)
		.attr("font-size", 12)
		.text("年份");
}

// 绘制柱状图
function drawBarChart(svg, data) {
	const margin = { top: 20, right: 30, bottom: 30, left: 50 };
	const width = 800 - margin.left - margin.right;
	const height = 500 - margin.top - margin.bottom;

	const xScale = d3
		.scaleBand()
		.domain(data.map((d) => d.year))
		.range([0, width])
		.padding(0.1);

	const yScale = d3
		.scaleLinear()
		.domain([
			d3.min(data, (d) => d.growthRate) * 1.05,
			d3.max(data, (d) => d.growthRate) * 1.05,
		])
		.range([height, 0]);

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left}, ${margin.top})`);

	g.selectAll("rect")
		.data(data)
		.join("rect")
		.attr("class", "bar")
		.attr("x", (d) => xScale(d.year))
		.attr("y", (d) => yScale(Math.max(0, d.growthRate)))
		.attr("width", xScale.bandwidth())
		.attr("height", (d) => Math.abs(yScale(d.growthRate) - yScale(0)))
		.attr("fill", (d) => (d.growthRate < 0 ? "red" : "lightgray"));

	// 添加X轴
	g.append("g")
		.attr("transform", `translate(0, ${height})`)
		.call(d3.axisBottom(xScale));

	// 添加Y轴
	g.append("g").call(d3.axisLeft(yScale));

	g.append("text")
		.attr("x", 10)
		.attr("y", 0)
		.attr("font-size", 12)
		.text("人口自然增长率（‰）");

	g.append("text")
		.attr("x", width - 4)
		.attr("y", height - 10)
		.attr("font-size", 12)
		.text("年份");
}

onMounted(async () => {
	const { lineData, barData } = await loadData();

	const svgBar = d3
		.select(barChart.value)
		.append("svg")
		.attr("width", 800)
		.attr("height", 500);

	const svgLine = d3
		.select(lineChart.value)
		.append("svg")
		.attr("width", 800)
		.attr("height", 500);

	drawLineChart(svgLine, lineData);
	drawBarChart(svgBar, barData);
});
</script>

<style scoped>

#tooltip {
    display: none;
	position: absolute;
	padding: 10px;
	background: lightgray;
	border: 1px solid gray;
	border-radius: 4px;
	pointer-events: none;
	z-index: 10;
	width: 300;
}
</style>
