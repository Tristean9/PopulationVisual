<template>
	<div class="container">
		<h1>中国人口数据总览</h1>
		<div class="text-container">
			<p>
				中国人口自然增长率在2000年后整体呈现逐渐降低趋势，在22年和23年出现负增长。22年的自然增长率为-0.6%，23年为-1.4%。其中，虽然有着2019年新冠肺炎疫情的影响，但总体趋势不变。
			</p>
		</div>
		<h2>人口增长情况（2000-2023）</h2>
		<div ref="chart"></div>
		<div id="tooltip">
			<p><span id="tooltip-year"></span></p>
			<p><span id="tooltip-population"></span></p>
		</div>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const chart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/人口增长情况（2000-2023）.csv`
	);

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

// 绘制合并图表
function drawChart(svg, lineData, barData) {
	const margin = { top: 20, right: 60, bottom: 50, left: 65 };
	const width = 850 - margin.left - margin.right;
	const height = 600 - margin.top - margin.bottom;

	const xScale = d3
		.scaleBand()
		.domain(lineData.map((d) => d.year))
		.range([0, width])
		.padding(0.1);

	// 左侧Y轴：人数
	const yLeftScale = d3
		.scaleLinear()
		.domain([
			d3.min(lineData, (d) => d.population) * 0.95,
			d3.max(lineData, (d) => d.population) * 1.05,
		])
		.range([height, 0]);

	// 右侧Y轴：增长率
	const yRightScale = d3
		.scaleLinear()
		.domain([
			d3.min(barData, (d) => d.growthRate) * 1.05,
			d3.max(barData, (d) => d.growthRate) * 1.05,
		])
		.range([height, 0]);

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left}, ${margin.top})`);

	// 绘制柱状图
	g.selectAll("rect")
		.data(barData)
		.join("rect")
		.attr("class", "bar")
		.attr("x", (d) => xScale(d.year))
		.attr("y", (d) => yRightScale(Math.max(0, d.growthRate)))
		.attr("width", xScale.bandwidth())
		.attr("height", (d) =>
			Math.abs(yRightScale(d.growthRate) - yRightScale(0))
		)
		.attr("fill", (d) => (d.growthRate < 0 ? "#a60126" : "lightgray"));

	// 绘制折线图
	const line = d3
		.line()
		.x((d) => xScale(d.year) + xScale.bandwidth() / 2)
		.y((d) => yLeftScale(d.population));

	g.append("path")
		.datum(lineData)
		.attr("fill", "none")
		.attr("stroke", "steelblue")
		.attr("stroke-width", 2)
		.attr("d", line);

	// 添加圆点
	g.selectAll("circle")
		.data(lineData)
		.join("circle")
		.attr("cx", (d) => xScale(d.year) + xScale.bandwidth() / 2)
		.attr("cy", (d) => yLeftScale(d.population))
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
		.attr("class", "x-axis")
		.attr("transform", `translate(0, ${height})`)
		.call(d3.axisBottom(xScale))
		.selectAll("text")
		.style("font-size", "10px");

	// 添加X轴标签
	g.append("text")
		.attr("x", width / 2)
		.attr("y", height + 40)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "13px")
		.text("年份");

	// 添加左侧Y轴
	g.append("g")
		.call(d3.axisLeft(yLeftScale))
		.selectAll("text")
		.style("font-size", "10px");

	g.append("text")
		.attr("transform", "rotate(-90)")
		.attr("x", -height / 2)
		.attr("y", -50)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "13px")
		.text("总人口 (万人)");

	// 添加右侧Y轴
	g.append("g")
		.attr("transform", `translate(${width}, 0)`)
		.call(d3.axisRight(yRightScale))
		.selectAll("text")
        .style("font-size", "10px");
        
	g.append("text")
		.attr("transform", "rotate(-90)")
		.attr("x", -height / 2)
		.attr("y", width + 40)
		.attr("fill", "black")
        .style("text-anchor", "middle")
        .style("font-size", "13px")
		.text("增长率 (%)");
}

onMounted(async () => {
	const { lineData, barData } = await loadData();

	const svg = d3
		.select(chart.value)
		.append("svg")
		.attr("width", 850)
		.attr("height", 600);

	drawChart(svg, lineData, barData);
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
	width: 200;
	height: 100px;
}
</style>
