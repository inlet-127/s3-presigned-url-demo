<template>
	<S3PresignedUrlSample msg="Welcome to Your Vue.js App" />
</template>

<script>
import S3PresignedUrlSample from "./components/S3PresignedUrlSample.vue";

export default {
	name: "App",
	components: {
		S3PresignedUrlSample,
	},
};
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
