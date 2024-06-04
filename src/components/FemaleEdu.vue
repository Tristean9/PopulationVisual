<template>
	<div class="row-container">
		<div class="chart-container">
			<h2>6岁及以上大专以上学历女性占比</h2>
			<div ref="chart" id="edu-chart"></div>
		</div>
		<div class="text-container">
			<p>
				<strong>教育程度提升的影响：</strong>
				随着女性受教育程度的提高，她们更倾向于追求个人事业和发展，延迟结婚和生育的可能性增加。受教育程度的提升使女性有更多的选择权和独立性，她们更加注重自身的成长和发展，而不是传统的婚姻和生育角色。
			</p>
		</div>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const width = 600;
const height = 400;
const margin = { top: 20, right: 30, bottom: 60, left: 65 };
const chart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(`${import.meta.env.BASE_URL}data/受教育程度.csv`);
	// console.log(data);

	const chartData = data
		.map((d) => ({
			year: d["年份"],
			value: +d["大专及以上女性人口占比"]*100,
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
		.selectAll("text")
		.attr("transform", "rotate(-45)") // 旋转角度
		.style("font-size", "10px")
		.style("text-anchor", "end"); // 文本对齐方式

	// 添加X轴标签
	g.append("text")
		.attr("class", "x-axis-label")
		.attr("x", (width - margin.left - margin.right) / 2)
		.attr("y", height - margin.bottom + 30) // 调整Y轴位置以确保标签在图表下方
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "13px")
		.text("年份");

	// 添加Y轴
	g.append("g")
		.attr("class", "y-axis")
		.call(d3.axisLeft(yScale))
		.selectAll("text")
		.style("font-size", "10px");

	// 添加Y轴标签
	g.append("text")
		.attr("class", "y-axis-label")
		.attr("transform", "rotate(-90)")
		.attr("x", -(height - margin.top - margin.bottom) / 2)
		.attr("y", -50)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "13px")
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
#edu-chart {
	margin: 0 2rem;
}
</style>
