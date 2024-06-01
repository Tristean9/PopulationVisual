<template>
	<div class="container">
		<div class="top">
			<div class="img-container">
				<img :src="imagePath" />
			</div>
			<div class="title-container">
				<h1 id="title">中国人口数据洞察报告</h1>
				<h2 id="subtitle">一份对人口负增长原因进行洞察的指南</h2>
			</div>
		</div>
		<div class="line"></div>
		<div class="bottom">
			<p>这是一份关于人口数据的报告</p>
			<p>
				回溯历史的长河，中国的人口变迁犹如一部波澜壮阔的史诗，诉说着民族的兴衰与文明的更迭。从古代的“人多力量大”到现代的“人口红利”，人口的增长与变迁始终与国家的命运紧密相连。
			</p>
			<p>
				新中国成立后，政府高度重视人口问题。从计划生育到全面二孩的放开政策经历了从紧到松的转变。然随着时代的进步，人口问题呈现出新的特点。老龄化、少子化等趋势加速到来，给社会经济发展带来深远影响。在这个历史性的转折点上，我们有必要重新审视中国的人口问题，深入剖析其发展规律和现状。
			</p>
		</div>
	</div>
</template>

<script setup>
const imagePath = `${import.meta.env.BASE_URL}introduction.png`;
</script>

<style scoped>
.container {
	display: flex;
	flex-direction: column;
    height: 800px;
}
.top {
	display: flex;
	height: 200px;
	margin: 0 20px;
    flex: 2
}
.title-container {
	color: #bd3124;
	flex: 2;
}

#title {
    font-size: 3rem;
    line-height: 36px;
    font-weight: bold;
    text-align: start;
	margin-left: 10%;
}
#subtitle {
    font-size: 1.8rem;
    line-height: 36px;
    text-align: end;
	margin-right: 10%;
}


.img-container {
	flex: 1;
}

.img-container img {
	width: 100%; /* 宽度充满容器 */
	max-width: 100%; /* 最大宽度不超过容器 */
	max-height: 100%; /* 最大高度不超过容器 */
	object-fit: contain; /* 保持图片的原始宽高比 */
}

.line {
	width: 100%;
	min-width: 10px;
	height: 30px;
	background-color: #bd3124;
}

.bottom {
    margin: 20px 20px;
    text-align: start;
    font-family: "PingFangSC-bold";
    line-height: 36px;
    font-weight: bold;
    flex: 2
}
.bottom p {
    font-size: 1.6rem;
}
</style>
