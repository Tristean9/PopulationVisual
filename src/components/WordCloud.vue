<template>
	<div class="container">
		<h1>时间不足和金钱压力成为“不愿生”的主要原因</h1>
		<h3>影响生育意愿的因素</h3>
		<div ref="chart"></div>
		<p>
			据人民智库中国公众生育观念调查报告（2023），收入压力、时间和精力不足、教育成本是育龄群体“不愿生”“不敢生”的首要因素
		</p>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";
import cloud from "d3-cloud";

const width = 800;
const height = 500;

const chart = ref(null);

async function loadData() {
	const response = await fetch(
		`${import.meta.env.BASE_URL}data/各因素对生育意愿的影响.json`
	);
	const data = await response.json();

	console.log(data);

	const chartData = Object.entries(data).map(([text, value]) => ({
		text,
		value,
	}));
	console.log(chartData);

	return chartData;
}
function drawChart(svg, data) {
	const layout = cloud()
		.size([width, height])
		.words(data)
		.padding(5)
		.rotate(() => (~~(Math.random() * 6) - 3) * 30)
		.fontSize((d) => d.value)
		.on("end", (words) => {
			svg.append("g")
				.attr("transform", `translate(${width / 2},${height / 2})`)
				.selectAll("text")
				.data(words)
				.enter()
				.append("text")
				.style("font-size", (d) => `${d.size}px`)
				.style("fill", (d, i) => d3.schemeCategory10[i % 10])
				.attr("text-anchor", "middle")
				.attr(
					"transform",
					(d) => `translate(${d.x},${d.y})rotate(${d.rotate})`
				)
				.text((d) => d.text);
		});

	layout.start();
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
