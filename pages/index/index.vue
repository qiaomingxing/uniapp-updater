<template>
	<view class="content">
		<view class="text-area">
			<button type="default" @click="checkUpdate">检测更新</button>
		</view>
		<app-updater ref="updater" :auto="true" @change="e=>isForceUpdate=e.isForceUpdate"></app-updater>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'Hello',
				isForceUpdate: false
			}
		},
		onLoad() {

		},
		onBackPress() {
			if (this.isForceUpdate) {
				return true;
			} 
			if (this.showDrawer) {
				this.showDrawer = false;
				return true;
			}
		},
		methods: {
			checkUpdate() {
				this.$refs.updater.checkUpdate()
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.text-area {
		display: flex;
		justify-content: center;
		padding: 50rpx 0;
	}
</style>
