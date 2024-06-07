<template>
	<div class="container">
		<female-edu />
		<gender-diff />
		<div class="text-container">
			<p>
				<strong>个体化动机的增强：</strong>
				随着社会的发展，个体化动机在人们的生活中变得越来越重要。女性的生育动机从社会化责任转向个体化动机，可能意味着女性更多地考虑到个人的发展、事业、生活质量等因素，而不仅仅是传统的社会责任。这导致了一些女性对于生育的愿望减少，从而影响了生育率的下降。
			</p>
		</div>
	</div>
</template>

<script setup>
import GenderDiff from "./GenderDiff.vue";
import FemaleEdu from "./FemaleEdu.vue";
</script>

<style scoped>
.text-container {
	text-align: start;
}
</style>
