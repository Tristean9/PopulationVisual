<template>
	<div class="container">
		<img :src="imagePath" />
	</div>
</template>

<script setup>
const imagePath = `${import.meta.env.BASE_URL}cover.png`;

console.log(imagePath);
</script>

<style scoped>
.container {
	display: inline-block;
	position: relative; /* 如果需要定位图片 */
}

.container img {
	width: 100%; /* 宽度充满容器 */
	height: auto; /* 高度自适应，保持图片原有宽高比 */
	/* min-height: 800px; */
}
</style>
