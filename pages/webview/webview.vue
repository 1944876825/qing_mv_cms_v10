<template>
	<view>
		<web-view :src="webUrl"></web-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				webUrl: ''
			};
		},
		onLoad(option) {
			this.webUrl = atob(option.webUrl)
		}
	}
</script>

<style lang="scss" scoped>

</style>
