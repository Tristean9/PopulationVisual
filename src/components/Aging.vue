<template>
	<div class="container">
		<h1>老龄化进程加快</h1>
		<div ref="chart"></div>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const width = 900;
const height = 400;
const chart = ref(null);

async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/不同年龄段人口占比.csv`,
		(d) => {
			d["0-14岁人口(万人)"] = +d["0-14岁人口(万人)"];
			d["15-64岁人口(万人)"] = +d["15-64岁人口(万人)"];
			d["65岁及以上人口(万人)"] = +d["65岁及以上人口(万人)"];
			return d;
		}
	);

	const chartData = data.map((d) => ({
		year: d["年份"],
		young:
			(d["0-14岁人口(万人)"] /
				(d["0-14岁人口(万人)"] +
					d["15-64岁人口(万人)"] +
					d["65岁及以上人口(万人)"])) *
			100,
		middle:
			(d["15-64岁人口(万人)"] /
				(d["0-14岁人口(万人)"] +
					d["15-64岁人口(万人)"] +
					d["65岁及以上人口(万人)"])) *
			100,
		old:
			(d["65岁及以上人口(万人)"] /
				(d["0-14岁人口(万人)"] +
					d["15-64岁人口(万人)"] +
					d["65岁及以上人口(万人)"])) *
			100,
	}));

	return chartData;
}

function drawChart(svg, data) {
	const margin = { top: 20, right: 150, bottom: 50, left: 60 };
	const chartWidth = width - margin.left - margin.right;
	const chartHeight = height - margin.top - margin.bottom;

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left},${margin.top})`);

	const x = d3
		.scaleBand()
		.domain(data.map((d) => d.year))
		.range([0, chartWidth])
		.padding(0.1);

	const y = d3.scaleLinear().domain([0, 100]).nice().range([chartHeight, 0]);

	const color = d3
		.scaleOrdinal()
		.domain(["young", "middle", "old"])
		.range(d3.schemeCategory10);

	const stack = d3.stack().keys(["young", "middle", "old"]);

	const series = stack(data);

	g.append("g")
		.selectAll("g")
		.data(series)
		.enter()
		.append("g")
		.attr("fill", (d) => color(d.key))
		.selectAll("rect")
		.data((d) => d)
		.enter()
		.append("rect")
		.attr("x", (d) => x(d.data.year))
		.attr("y", (d) => y(d[1]))
		.attr("height", (d) => y(d[0]) - y(d[1]))
		.attr("width", x.bandwidth());

	g.append("g")
		.attr("class", "x-axis")
		.attr("transform", `translate(0,${chartHeight})`)
		.call(d3.axisBottom(x))
		.append("text")
		.attr("y", 35)
		.attr("x", chartWidth / 2)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.text("年份");

	g.append("g")
		.attr("class", "y-axis")
		.call(d3.axisLeft(y))
		.append("text")
		.attr("transform", "rotate(-90)")
		.attr("y", -45)
		.attr("x", -chartHeight / 2)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.text("占比(%)");

	// 添加图例
	const legend = svg
		.append("g")
		.attr("transform", `translate(${margin.left + chartWidth + 20},${margin.top})`);

    const legendData = ["young", "middle", "old"];
    const temp = {
        "young": "0-14岁",
        "middle": "15-64岁",
        "old": "65岁及以上"
    }
	legendData.forEach((key, i) => {
		const legendRow = legend
			.append("g")
			.attr("transform", `translate(0, ${i * 30})`);

		legendRow
			.append("rect")
			.attr("width", 20)
			.attr("height", 20)
			.attr("fill", color(key));

		legendRow
			.append("text")
			.attr("x", 25)
			.attr("y", 15)
			.attr("text-anchor", "start")
			.style("text-transform", "capitalize")
			.text(temp[key]);
	});
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
.container {
	width: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
}
</style>
