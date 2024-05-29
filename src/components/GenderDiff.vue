<template>
	<div class="container">
		<h3>不同性别对各个观点的看法</h3>
		<div ref="chart"></div>
	</div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as d3 from "d3";

const width = 1200;
const height = 400;

const chart = ref(null);

// 加载数据
async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/男女性观念差别.csv`
	);

	// console.log(data);

	const chartData = data
		.map((d) => ({
			viewpoint: d["观点"],
			male: +d["男性"],
			female: +d["女性"],
		}))
		.reverse();

	// console.log(chartData);

	return chartData;
}

function drawChart(svg, data) {
	const start = 700;
	const barHeight = 40;
	const gap = 20;
	const barMargin = 50;

	// 绑定数据到男性矩形
	svg.selectAll(".bar.female")
		.data(data)
		.enter()
		.append("rect")
		.attr("class", "bar female")
		.attr("y", (d, i) => i * barMargin + gap) // 每个矩形间隔为50，起始间隔为10
		.attr("x", (d) => start - d.female * 5)
		.attr("width", (d) => d.female * 5) // 使用宽度*10像素值
		.attr("height", barHeight) // 设置固定高度
		.attr("fill", "pink");

	// 绑定数据到女性矩形
	svg.selectAll(".bar.male")
		.data(data)
		.enter()
		.append("rect")
		.attr("class", "bar male")
		.attr("y", (d, i) => i * barMargin + gap) // 每个矩形间隔为50，起始间隔为10
		.attr("x", (d) => start) // 从男性矩形的结束位置开始
		.attr("width", (d) => d.male * 5) // 使用宽度*10像素值
		.attr("height", barHeight) // 设置固定高度
		.attr("fill", "#ADD8E6");

	// 添加建筑物名称标签
	svg.selectAll(".label")
		.data(data)
		.enter()
		.append("text")
		.attr("class", "label")
		.attr("y", (d, i) => i * barMargin + gap + barHeight / 2) // 向下移动
		.attr("x", 250) //
		.attr("text-anchor", "end")
		.text((d) => d.viewpoint);

	// 添加x轴
	const xScale = d3
		.scaleLinear()
		.domain([-80, 80])
		.range([start - 400, start + 400]); // 范围长度

	const xAxis = d3.axisBottom(xScale).tickFormat((d) => Math.abs(d)); // 将负值转换为正值

	svg.append("g")
		.attr("class", "x-axis")
		.attr("transform", `translate(0, ${data.length * barMargin + gap})`) // 将x轴放在条形图下方
		.call(xAxis);

	svg.append("text")
		.attr("x", start)
		.attr(
			"transform",
			`translate(0, ${data.length * barMargin + gap + 35})`
		) // 将x轴放在条形图下方
		.attr("text-anchor", "middle")
		.text("百分比");

	// 图例矩形的宽度
	const legendWidth = 40;
	// 图例矩形的高度
    const legendHeight = 15;
    // 图例的起始位置
    const legendStart = [width - 150, 20]
    const legendGap = 30;

	// 添加图例
	svg.append("rect")
		.attr("x", legendStart[0])
		.attr("y", legendStart[1])
		.attr("width", legendWidth)
		.attr("height", legendHeight)
		.attr("fill", "pink");

	svg.append("text").attr("x", legendStart[0] + 50).attr("y", legendStart[1] + 13).text("女性");

	svg.append("rect")
		.attr("x", legendStart[0])
		.attr("y", legendStart[1]+legendGap)
		.attr("width", legendWidth)
		.attr("height", legendHeight)
		.attr("fill", "#ADD8E6");

	svg.append("text").attr("x", legendStart[0] + 50).attr("y", legendStart[1] + legendGap+ 13).text("男性");
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
