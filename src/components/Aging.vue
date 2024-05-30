<template>
	<div class="container">
		<h1>老龄化进程加快</h1>
		<div ref="chart"></div>
		<div class="text-container">
			<p>
				中国自改革开放以来，人口结构逐渐发生了重大变化。在人口红利时期，中国以其庞大的劳动力资源为支撑，取得了长期的经济增长。然而，随着时间的推移，人口结构发生了根本性的转变，老龄化问题日益突显。
			</p>
			<p>
				中国于2022年的“65岁及以上”人口已达到20978万人，占比14.9%，正式标志着中国步入“中度老龄化”社会。这一数字的迅速增长以及老龄化进程的加速，给中国社会经济带来了严峻挑战。
			</p>
			<p>
				造成这一现象的原因多方面，收入压力、时间和精力不足以及教育成本等因素导致了育龄群体“不愿生”“不敢生”，进而使得人口自然增长率持续下降。同时，随着社会发展，个体化动机越来越重要，女性生育动机转向个体化，导致生育率降低。教育程度提升也是影响生育率的重要因素，随着女性受教育程度的提高，延迟结婚和生育的可能性增加。
			</p>
			<p>
				尽管一些地区出台了针对多子女家庭的优惠政策，但从数据上看，这些政策的效果有限。独生子女政策和全面二孩政策虽然在一定程度上影响了人口增长，但无法阻止老龄化进程的加速。事实上，中国人口红利已成为历史，老龄化问题已成为当务之急。
			</p>
			<p>
				因此，中国政府需要加大力度，采取有效措施，应对人口老龄化和生育率下降的问题。这可能包括优化生育政策，提高生育率；加强宣传与引导，克服青年人恐育思想；第三，关注女性群体利益，切实减轻职场女性家庭与事业选择两难问题。
			</p>
		</div>
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
		.attr(
			"transform",
			`translate(${margin.left + chartWidth + 20},${margin.top})`
		);

	const legendData = ["young", "middle", "old"];
	const temp = {
		young: "0-14岁",
		middle: "15-64岁",
		old: "65岁及以上",
	};
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
