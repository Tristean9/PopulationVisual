<template>
	<div class="container">
		<h1>时间不足和金钱压力成为“不愿生”的主要原因</h1>
		<h3>各因素对生育意愿的影响</h3>
		<div ref="chart"></div>
		<p>
			据人民智库中国公众生育观念调查报告（2023），收入压力、时间和精力不足、教育成本是育龄群体“不愿生”“不敢生”的首要因素
		</p>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const width = 800;
const height = 500;

const chart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/各因素对生育意愿的影响.csv`
	);

	// console.log(data);

	const chartData = data.map((d) => ({
		reason: d["因素"],
		value: +d["百分比"],
	}));

	// console.log(chartData);

	return chartData;
}

function drawChart(svg, data) {
	const start = 150;
	// 绑定数据到矩形
	svg.selectAll(".bar")
		.data(data)
		.enter()
		.append("rect")
		.attr("class", "bar")
		.attr("y", (d, i) => i * 50 + 10) // 每个矩形宽度为50，间隔为10
		.attr("x", (d) => start)
		.attr("width", (d) => d.value * 10) // 使用宽度减去宽度像素值
		.attr("height", 20) // 设置固定高度
		.attr("fill", "steelblue");

	// 添加原因
	svg.selectAll(".label")
		.data(data)
		.enter()
		.append("text")
		.attr("class", "label")
		.attr("y", (d, i) => i * 50 + 25) // 向下移动
		.attr("x", start - 10) //
		.attr("text-anchor", "end")
		.text((d) => d.reason);

	// 添加建筑物高度标签
	svg.selectAll(".height")
		.data(data)
		.enter()
		.append("text")
		.attr("class", "height")
		.attr("y", (d, i) => i * 50 + 25) // 向下移动
		.attr("x", (d) => d.value * 10 + start) // 放置在矩形右侧
		.attr("fill", "black") // 设置字体为黑色
		.attr("font-size", "14px")
		.text((d) => `${d.value}%`);
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

<style scoped></style>
