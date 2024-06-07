<template>
	<div class="container">
		<h1>相关政策总览</h1>
		<div class="text-container">
			<p>
				虽然2022年后，不同地区出台了很对针对多子女家庭的优惠政策。但从数据角度可以看出效果微小。对人口影响较为严重的仍然是整体的宏观调控。独生子女政策的影响十分巨大。开放二胎的政策虽使得人口数量短暂回升，但仍无法阻止减少的大趋势问题。对多子女家庭的地区优惠政策，整体在2021年后进行实施。但从结果来看收效甚微。
			</p>
		</div>
		<div class="chart-container">
			<h2>部分省份人口自然增长率对比</h2>
			<div ref="chart"></div>
		</div>
		<div ref="tooltipLeft" id="tooltip-left"  class="tooltip left" v-show="showTooltip === 'left'">
			<p>
				东北地区的计划生育政策比其他地区更为严格，这主要是因为当时东北三省的经济发展速度较快，而南方地区还在起步阶段。因此，东北地区的计划生育政策执行力度大，导致了东北地区的生育率相对较低。
			</p>
		</div>
		<div ref="tooltipMid" id="tooltip-mid"  class="tooltip mid" v-show="showTooltip === 'mid'">
			<p>全面废除独生子女政策，第二年的人口增长率有明显提高</p>
		</div>
		<div ref="tooltipRight" id="tooltip-right" class="tooltip right" v-show="showTooltip === 'right'">
			<p>中央政府开放三胎政策，但对人口增长并无帮助</p>
		</div>
	</div>
</template>

<script setup>
import { nextTick, onMounted, ref } from "vue";
import * as d3 from "d3";

const showTooltip = ref(null);

const width = 1000;
const height = 700; // 增加SVG高度
const chartHeight = 600; // 图表的实际高度
const margin = { top: 20, right: 100, bottom: 60, left: 60 }; // 增加底部的margin
const chart = ref(null);
const tooltipLeft = ref(null);
const tooltipMid = ref(null);
const tooltipRight = ref(null);

// 显示或隐藏气泡框
const toggleTooltip = (position) => {
	showTooltip.value = position;
};

// 加载数据
async function loadData() {
	const data = await d3.csv(
		`${import.meta.env.BASE_URL}data/若干省生育率.csv`
	);

	let all_data = [];
	for (let i = 0; i < data.length; i++) {
		let temp = {};
		for (let key in data[i]) {
			if (key === "省份") {
				temp["province"] = data[i][key];
			} else {
				if (temp["oneData"] == undefined) {
					temp["oneData"] = [{ year: key, value: +data[i][key] }];
				} else {
					temp["oneData"].push({ year: key, value: +data[i][key] });
				}
			}
		}
		all_data.push(temp);
	}
	return all_data;
}

// 在SVG内部添加按钮
function addButtons(svg) {
	const buttonData = [
		{ label: "2016年以前：计划生育时期", position: "left" },
		{ label: "2016年：废除独生子女政策", position: "mid" },
		{ label: "2021年：开放三胎政策", position: "right" },
	];

	const buttonWidth = 300;
	const buttonHeight = 45;
	const buttonSpacing = 0;
	const totalButtonWidth =
		buttonData.length * (buttonWidth + buttonSpacing) - buttonSpacing;
	const buttonStartX = (width - totalButtonWidth) / 2;

	svg.selectAll(".button")
		.data(buttonData)
		.enter()
		.append("foreignObject")
		.attr("x", (d, i) => buttonStartX + i * (buttonWidth + buttonSpacing))
		.attr("y", height - 100) // 确保按钮在标签下方
		.attr("width", buttonWidth)
        .attr("height", buttonHeight)
		.append("xhtml:button")
		.attr("class", "chart-button")
		.on("click", (event, d) => {
			toggleTooltip(d.position);
			const tooltipElements = {
				left: tooltipLeft.value,
				mid: tooltipMid.value,
				right: tooltipRight.value,
			};

			const tooltipPositions = {
				left: {
					left: event.pageX - 100 +  "px",
					top: event.pageY - 600 + "px",
				},
				mid: {
					left: event.pageX + 90+ "px",
					top: event.pageY - 600 + "px",
				},
				right: {
					left: event.pageX + 50+ "px",
					top: event.pageY - 230 + "px",
				},
			};

			Object.keys(tooltipElements).forEach((key) => {
				const tooltip = tooltipElements[key];
				if (key === d.position) {
					tooltip.style.display = "block";
					tooltip.style.left = tooltipPositions[key].left;
					tooltip.style.top = tooltipPositions[key].top; // 设定单独的上方位置
				} else {
					tooltip.style.display = "none";
				}
			});
		})
		.text((d) => d.label);
}

// 绘制折线图
function drawChart(svg, data) {
	const years = data[0].oneData.map((d) => d.year);
	const provinces = data.map((d) => d.province);

	const xScale = d3
		.scaleBand()
		.domain(years)
		.range([0, width - margin.left - margin.right])
		.padding(0.1);

	const yScale = d3
		.scaleLinear()
		.domain([
			d3.min(data, (c) => d3.min(c.oneData, (d) => d.value)) * 1.05,
			d3.max(data, (c) => d3.max(c.oneData, (d) => d.value)) * 1.05,
		])
		.range([chartHeight - margin.top - margin.bottom, 0]);

	const color = d3.scaleOrdinal(d3.schemeCategory10).domain(provinces);

	const g = svg
		.append("g")
		.attr("transform", `translate(${margin.left},${margin.top})`);

	const line = d3
		.line()
		.x((d) => xScale(d.year) + xScale.bandwidth() / 2)
		.y((d) => yScale(d.value));

	const province = g
		.selectAll(".province")
		.data(data)
		.enter()
		.append("g")
		.attr("class", "province");

	province
		.append("path")
		.attr("class", "line")
		.attr("d", (d) => line(d.oneData))
		.style("stroke", (d) => color(d.province))
		.style("fill", "none")
		.style("stroke-width", 2);

	// 添加X轴
	g.append("g")
		.attr("class", "x-axis")
		.attr(
			"transform",
			`translate(0, ${chartHeight - margin.top - margin.bottom})`
		)
		.call(d3.axisBottom(xScale))
		.selectAll("text")
		.style("font-size", "12px");

	// 添加X轴标签
	g.append("text")
		.attr("class", "x-axis-label")
		.attr("x", (width - margin.left - margin.right) / 2)
		.attr("y", chartHeight - margin.top - margin.bottom + 40) // 在X轴下方
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "15px")
		.text("年份");

	// 添加Y轴
	g.append("g")
		.attr("class", "y-axis")
		.call(d3.axisLeft(yScale))
		.selectAll("text")
		.style("font-size", "12px");

	// 添加Y轴标签
	g.append("text")
		.attr("transform", "rotate(-90)")
		.attr("x", -(chartHeight - margin.top - margin.bottom) / 2)
		.attr("y", -40)
		.attr("fill", "black")
		.style("text-anchor", "middle")
		.style("font-size", "15px")
		.text("人口自然增长率 (‰)");

	// 添加图例
	const legend = svg
		.append("g")
		.attr(
			"transform",
			`translate(${width - margin.right + 10},${margin.top})`
		);

	legend
		.selectAll("rect")
		.data(provinces)
		.enter()
		.append("rect")
		.attr("x", 0)
		.attr("y", (d, i) => i * 20)
		.attr("width", 30)
		.attr("height", 4)
		.attr("fill", (d) => color(d));

	legend
		.selectAll("text")
		.data(provinces)
		.enter()
		.append("text")
		.attr("x", 40)
		.attr("y", (d, i) => i * 20 + 9)
		.text((d) => d);

	// 添加竖直虚线
	const xPosition2016 = xScale("2016");
	g.append("line")
		.attr("x1", xPosition2016 + 12)
		.attr("y1", margin.top)
		.attr("x2", xPosition2016 + 12)
		.attr("y2", chartHeight - margin.bottom - 20)
		.attr("stroke", "gray")
		.attr("stroke-dasharray", "4");

	// 添加竖直虚线
	const xPosition2021 = xScale("2021");
	g.append("line")
		.attr("x1", xPosition2021 + 12)
		.attr("y1", margin.top)
		.attr("x2", xPosition2021 + 12)
		.attr("y2", chartHeight - margin.bottom - 20)
		.attr("stroke", "gray")
		.attr("stroke-dasharray", "4");

	// 在SVG内部添加按钮
	addButtons(svg);
}

onMounted(async () => {
	const data = await loadData();

	const svg = d3
		.select(chart.value)
		.append("svg")
		.attr("width", width)
		.attr("height", height);
	nextTick(() => {
		drawChart(svg, data);
	});
});
</script>
<style>
.chart-button {
	margin: 0 10px;
	padding: 10px 20px;
	cursor: pointer;
}

.tooltip {
	position: absolute;
	background-color: #f9f9f9;
	border: 1px solid #ccc;
	border-radius: 4px;
	padding: 0 10px;
	z-index: 10;

	text-align: start;
	font-size: 14px;
}
.left {
	display: block;
	max-width: 400px;
}
.mid {
	display: block;
	max-width: 200px;
}
.right {
	display: block;
	max-width: 200px;
}
</style>
