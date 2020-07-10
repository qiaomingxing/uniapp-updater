<template>
	<view @touchmove.stop.prevent>
		<view class="updater-modal-box" :class="[modalVisible?'updater-modal-normal':'updater-modal-scale',modalVisible?'updater-modal-show':'']">
			<view>
				<view class="updater-modal-title">版本更新</view>
				<view class="updater-modal-content">
					<view class="is-text-left">
						<view v-for="(item, index) in contentList" :key="index">{{ item }}</view>
					</view>
				</view>
				<view class="updater-modalBtn-box">
					<button v-if="!isForceUpdate" class="updater-modal-btn updater-gray-outline" hover-class="updater-outline-hover"
					 @tap="closeModal">稍后更新</button>
					<button class="updater-modal-btn updater-primary" hover-class="updater-primary-hover" @tap="confirmModal">立即更新</button>
				</view>
			</view>
		</view>
		<view class="updater-modal-mask" :class="[modalVisible?'updater-mask-show':'']" @tap="maskClick"></view>
	</view>
</template>

<script>
	export default {
		name: 'AppUpdater',
		components: {},
		props: {
			// 是否自动检测更新
			auto: {
				required: false,
				type: Boolean,
				default: false
			},
			// 检测更新Url
			checkUrl: {
				required: false,
				type: String,
				default: 'http://api.update.com'
			},
			//点击遮罩 是否可关闭
			maskClosable: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				modalVisible: false,
				downloadUrl: '',
				contentList: [],
				versionCode: '',
				isForceUpdate: 0
			};
		},
		mounted() {
			if (this.auto && !this.isCheckedUpdate) {
				this.checkUpdate();
			}
		},
		methods: {
			// 遮层点击事件
			maskClick() {
				if (!this.maskClosable) return;
				this.closeModal()
			},
			// 显示弹窗
			showModal() {
				this.modalVisible = true;
			},
			// 隐藏更新弹窗
			closeModal() {
				this.modalVisible = false;
			},
			// 确定更新App
			confirmModal() {
				// #ifdef APP-PLUS
				plus.runtime.openURL(this.downloadUrl);
				// #endif
				// #ifndef APP-PLUS
				window.open(this.downloadUrl)
				// #endif
			},
			// 检测更新
			checkUpdate(showtip = false) {
				let curVersion = 1
				// #ifdef APP-PLUS
				curVersion = plus.runtime.versionCode;
				// #endif
				this.checkVersion(curVersion).then(res => {
					if (res.code === 200) {
						this.updateData = res.data
						const {
							versionName,
							versionCode,
							versionDownloadUrl,
							versionDesc,
							isForceUpdate
						} = res.data
						this.versionCode = versionCode;
						this.downloadUrl = versionDownloadUrl;
						this.isForceUpdate = isForceUpdate
						if (versionDesc) {
							this.contentList = versionDesc.split('|');
						} else {
							this.contentList = ['发现新版本', `${versionName}`]
						}
						this.$emit('change', res.data)
						this.showModal();
					} else if (showtip) {
						this.$tips.toast('没有新版本')
					}
				});
			},
			// 检查版本更新，调用服务器接口
			checkVersion(versionCode) {
				return new Promise((resolve, reject) => {
					const result = {
						code: 200,
						data: {
							versionName: 'V1.0.0',
							versionCode: 2,
							versionDownloadUrl: 'http://www.baidu.com',
							versionDesc: '检测到您有新版本可以更新V1.0.0|1. 优化登录流程|2. 优化注册流程',
							isForceUpdate: true
						},
						msg: 'success'
					}
					resolve(result)
				})
			}
		}
	};
</script>

<style lang="scss">
	.updater-modal-box {
		position: fixed;
		left: 50%;
		top: 50%;
		margin: auto;
		background: #fff;
		z-index: 9999998;
		transition: all 0.3s ease-in-out;
		opacity: 0;
		width: 84%;
		padding: 40rpx 64rpx;
		border-radius: 24rpx;
		box-sizing: border-box;
		visibility: hidden;
	}

	.updater-modal-scale {
		transform: translate(-50%, -50%) scale(0);
	}

	.updater-modal-normal {
		transform: translate(-50%, -50%) scale(1);
	}

	.updater-modal-show {
		opacity: 1;
		visibility: visible;
	}

	.updater-modal-mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.6);
		z-index: 9999996;
		transition: all 0.3s ease-in-out;
		opacity: 0;
		visibility: hidden;
	}

	.updater-mask-show {
		visibility: visible;
		opacity: 1;
	}

	.updater-modal-title {
		text-align: center;
		font-size: 34rpx;
		color: #333;
		padding: 10rpx 0;
		font-weight: bold;
	}

	.updater-modal-content {
		text-align: center;
		color: #999;
		font-size: 28rpx;
		padding-top: 20rpx;
		padding-bottom: 40rpx;
	}

	.updater-modalBtn-box {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-between
	}

	.updater-flex-column {
		flex-direction: column;
	}

	.updater-modal-btn {
		flex: 1;
		width: 46%;
		height: 76rpx;
		line-height: 76rpx;
		position: relative;
		border-radius: 10rpx;
		font-size: 28rpx;
		overflow: visible;
		margin-left: 0;
		margin-right: 0;
	}

	.updater-modal-btn+.updater-modal-btn {
		margin-left: 30rpx;
	}


	.updater-modal-btn::after {
		content: "";
		position: absolute;
		width: 200%;
		height: 200%;
		-webkit-transform-origin: 0 0;
		transform-origin: 0 0;
		-webkit-transform: scale(0.5, 0.5);
		transform: scale(0.5, 0.5);
		left: 0;
		top: 0;
		border-radius: 20rpx;
	}

	.updater-primary {
		background: #16a98c;
		color: #fff;
	}

	.updater-primary-hover {
		background: rgba(22, 169, 140, 0.7);
		color: #e5e5e5;
	}

	.updater-primary-outline {
		color: #16a98c;
		background: none;
	}

	.updater-primary-outline::after {
		border: 1px solid #16a98c;
	}

	.updater-danger {
		background: #ed3f14;
		color: #fff;
	}

	.updater-danger-hover {
		background: #d53912;
		color: #e5e5e5;
	}

	.updater-danger-outline {
		color: #ed3f14;
		background: none;
	}

	.updater-danger-outline::after {
		border: 1px solid #ed3f14;
	}

	.updater-red {
		background: #e41f19;
		color: #fff;
	}

	.updater-red-hover {
		background: #c51a15;
		color: #e5e5e5;
	}

	.updater-red-outline {
		color: #e41f19;
		background: none;
	}

	.updater-red-outline::after {
		border: 1px solid #e41f19;
	}

	.updater-warning {
		background: #ff7900;
		color: #fff;
	}

	.updater-warning-hover {
		background: #e56d00;
		color: #e5e5e5;
	}

	.updater-warning-outline {
		color: #ff7900;
		background: none;
	}

	.updater-warning-outline::after {
		border: 1px solid #ff7900;
	}

	.updater-green {
		background: #19be6b;
		color: #fff;
	}

	.updater-green-hover {
		background: #16ab60;
		color: #e5e5e5;
	}

	.updater-green-outline {
		color: #19be6b;
		background: none;
	}

	.updater-green-outline::after {
		border: 1px solid #19be6b;
	}

	.updater-white {
		background: #fff;
		color: #333;
	}

	.updater-white-hover {
		background: #f7f7f9;
		color: #666;
	}

	.updater-white-outline {
		color: #333;
		background: none;
	}

	.updater-white-outline::after {
		border: 1px solid #333;
	}

	.updater-gray {
		background: #ededed;
		color: #999;
	}

	.updater-gray-hover {
		background: #d5d5d5;
		color: #898989;
	}

	.updater-gray-outline {
		color: #999;
		background: none;
	}

	.updater-gray-outline::after {
		border: 1px solid #999;
	}

	.updater-outline-hover {
		opacity: 0.6;
	}

	.updater-circle-btn {
		border-radius: 40rpx !important;
	}

	.updater-circle-btn::after {
		border-radius: 80rpx !important;
	}

	.is-text-left {
		text-align: left;
	}
</style>
