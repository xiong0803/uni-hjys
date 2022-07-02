<template>
	<view class="content">

		<view class="bigbox">
			<view class="Box">
				<view class="rotateBox" :style="'background-image: url('+info.image+');background-size: 100% 100%'">
					<image class="image" src="../../static/img/index/bj3.png"></image>
				</view>
			</view>
			<view class="Box1">
				<image class="img" src="../../static/img/index/b1.png" mode=""></image>
				<view class="center">
					<view class="goodsName">{{info.name}}</view>
					<view class="flexBox LimitBox">
						<view class="Limit">限量</view>
						<view class="stock">{{info.surplus}}份</view>
					</view>
				</view>
				<image class="img" src="../../static/img/index/b2.png" mode=""></image>
			</view>
		</view>

		<view class="type1" v-if="type == 1">
			<view class="msgBox">
				<!-- <view class="goodsName">{{info.name}}</view> -->
				<view class="flex_bt">
					<view class="priceBox">当前价: <text>¥{{info.price}}</text> </view>
					<view class="category_name">{{info.goods_category_name}} </view>
				</view>

				<view class="describe">{{info.title}}</view>
			</view>
			<view class="goodsinfo">
				<view class="iptBox  flexBox">
					<view class="label">创作者</view>
					<view class="center">{{info.creator}}
						<!-- <image class="copy" src="../../static/img/my/copy.png" mode=""></image> -->
					</view>
				</view>
				<view class="iptBox flexBox">
					<view class="label">拥有者</view>
					<view class="center">
						<text>{{info.owner}}</text>
						<image class="copy" @tap="copy(info.owner)" src="../../static/img/my/copy.png" mode=""></image>
					</view>
				</view>
				<view class="iptBox iptBox1 flexBox">
					<view class="label">铸造平台</view>
					<view class="center">{{info.casting_name}}</view>
				</view>
				<view class="iptBox iptBox1 flexBox">
					<view class="label">铸造时间</view>
					<view class="center">{{info.casting_time}}</view>
				</view>
				<!-- <view class="iptBox iptBox1 flexBox">
					<view class="label">商品ID</view>
					<view class="center">
						{{info.asset_id}}
						<image v-if="info.asset_id" class="copy" @tap="copy(info.asset_id)"
							src="../../static/img/my/copy.png" mode=""></image>
					</view>
				</view> -->
				<!-- <view class="iptBox iptBox1 flexBox">
					<view class="label">TokenID</view>
					<view class="center">
						{{info.token_id}}
						<image v-if="info.token_id" class="copy" @tap="copy(info.token_id)"
							src="../../static/img/my/copy.png" mode=""></image>
					</view>
				</view> -->
				<!-- <view class="iptBox iptBox1 flexBox">
					<view class="label">合约地址</view>
					<view class="center">
						{{info.contract_address}}
						<image class="copy" @tap="copy(info.contract_address)" src="../../static/img/my/copy.png"
							mode=""></image>
					</view>
				</view> -->
				<!-- <view class="iptBox iptBox1 flexBox">
					<view class="label">区块链</view>
					<view class="center">
						{{info.blockchain}}
						<image class="copy" @tap="copy(info.blockchain)" src="../../static/img/my/copy.png" mode="">
						</image>
					</view>
				</view> -->
			</view>
			<view class="descBox" v-if="info.content">
				<view class="item">藏品介绍</view>
				<view class="desinfo" v-html="util.checkImg(info.content)"></view>
			</view>
			<view class="footerBox flex_bt">
				<view class="price">¥{{info.price}}</view>
				<view class="subBtn " @tap="pay()" v-if="flag=='false'">立即购买</view>
				<view class="subBtn subnrn1" v-else @click="toast1()">立即购买</view>
			</view>
		</view>

		<view class="type2" v-if="type == 2">
			<view class="msgBox">
				<!-- <view class="goodsName">{{info.name}}</view> -->
				<view class="flex_bt">
					<view class="priceBox">当前价: <text>¥{{info.price}}</text> </view>
					<view class="number">本期共{{info.goods_count}}款藏品</view>
				</view>
				<view class="describe">{{info.title}}</view>
				<view class="rule">
					<view class="rule1">抢购预约规则:</view>
					<view class="" v-html="buy_content"></view>
				</view>
			</view>
			<view class="goodsList">
				<view class="goodsItem" v-for="(item, index) in goodsList" :key="index">
					<image class="goodsImg" :src="item.image" mode=""></image>
					<view v-if="item.surplus == 0" class="mask flex_column">
						<image class="maskImg" src="../../static/img/index/no.png" mode=""></image>
					</view>
					<view class="goodsinfo">
						<view class="goodsName">{{item.name}}</view>
						<view class="flexBox">
							<view class="price">¥<text>{{item.price}}</text></view>
							<image v-if="item.coupon_id" class="coupon" src="../../static/img/index/coupon.png" mode="">
							</image>
							<view class="count">剩余{{item.surplus}}件</view>
						</view>
					</view>
				</view>
			</view>
			<view class="footerBox flex_bt">
				<view class="count">{{info.goods_count}}款藏品可购买</view>
				<view class="subBtn" @tap="pay()">立即购买</view>
			</view>
		</view>
		<view class="tech-tip">
			<image src="../../static/img/tect-tip.png" mode=""></image>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				goodsId: '',
				banner: {},
				info: {},
				type: '', //单品 多品
				goodsList: [],
				buy_content: '',
				fullscreen: true,
				flag: 'false',
				text: '',
				rzflag: false,
				payflag: 'false',
				tqDate: 0,
			}
		},
		onLoad(e) {
			this.goodsId = e.goodsId;
			this.payflag = e.flag;
			this.flag = e.flag;
			this.text = e.text;
			this.status = e.status;
		},
		onShow() {
			this.rz();
			this.getData();
		},
		methods: {
			async rz() {
				try {
					let res = await this.$http.get('user/userInfo');
					console.log(res);
					if (res.code == 1) {
						if (res.data.name) {
							this.rzflag = true;
						} else {
							this.rzflag = false;
						}
					}
				} catch (e) {}

			},
			async statusJudge() {
				try {
					let res = await this.$http.post("index/yxg", {
						goods_id: this.goodsId
					});
					this.tqDate = res.times;
					this.appointAndPay();
				} catch (e) {
					//TODO handle the exception
				}
			},
			appointAndPay() {
				let cpdate = new Date(this.info.start_time).getTime();
				let dqdate = new Date().getTime();
				let sub = (this.tqDate - 0) * 60 * 1000;
				if (dqdate) {
					if (dqdate + sub >= cpdate) {
						this.flag = "false";
					} else {
						this.flag = "true";
					}
				}
				console.log(this.flag);
			},
			pay() {
				if (this.rzflag) {
					this.go('orderMakeSure?goodsId=' + this.goodsId);
				} else {
					this.toast("未实名认证，请先实名认证");
					setTimeout(() => {
						this.go('/pages/my/authentication');
					}, 500)

				}

			},
			toast1() {
				if (this.rzflag) {
					this.toast(this.text);
				} else {
					this.toast("未实名认证，请先实名认证");
					setTimeout(() => {
						this.go('/pages/my/authentication');
					}, 500)
				}
			},
			getData() {
				this.$http.get('goods/goodsDetail', {
					id: this.goodsId
				}).then(res => {
					if (res.code == 1) {
						this.info = res.data;
						this.type = res.data.type;
						if (this.info.surplus == 0) {
							this.flag = "true";
						} else {
							if (this.status == 2) {
								this.statusJudge();
							}
						}
					}
				})
			},
			getConfig() {
				this.$http.post('login/config').then(res => {
					if (res.code == 1) {
						this.buy_content = res.data.buy_content
					}
				})
			}
		}
	}
</script>
<style lang="scss" scoped>
	@keyframes myfirst {
		0% {
			transform: rotatey(0deg)
		}

		25% {
			transform: rotatey(90deg)
		}

		50% {
			transform: rotatey(180deg)
		}

		75% {
			transform: rotatey(270deg)
		}

		100% {
			transform: rotatey(360deg)
		}
	}

	.smpop {
		width: 600rpx;
		height: auto;

		.smpop-title {
			margin-top: 20rpx;
			margin-bottom: 20rpx;
			font-size: 30rpx;
			color: #000;
			font-weight: bold;
			text-align: center;
		}

		.cppop-cons {
			margin-top: 10rpx;
			width: calc(100% - 60rpx);
			height: 500rpx;
			padding: 0 30rpx;
		}

		.smpop-text {
			font-size: 24rpx;
			line-height: 40rpx;
			padding: 0 30rpx;
		}

		.smpop-btn {
			margin: 30rpx auto;
			width: 540rpx;
			height: 80rpx;
			background-color: #000;
			border-radius: 20rpx;
			font-size: 30rpx;
			color: #FFFFFF;
			text-align: center;
			line-height: 80rpx;
		}
	}

	.yqbox {
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.6);
		position: fixed;
		top: 0;
		z-index: 9999;

		.yqbox-clear {
			width: 100%;
			height: 52rpx;

			.yqbox-c {
				float: right;
			}
		}

		.yqbox-con {
			width: 562rpx;
			height: 1000rpx;
			position: absolute;
			top: 0;
			right: 0;
			bottom: 0;
			left: 0;
			margin: auto;

			.yqbox-cp {
				background-color: #000000;
				border-radius: 20rpx;
				padding-bottom: 30rpx;

				.yqbox-cp-img {
					display: block;
					width: 562rpx;
					height: 562rpx;
					border-radius: 20rpx;
				}

				.yqboxCp {
					width: 100%;
					height: auto;
					display: flex;

					.yqboxCp-left {
						width: 300rpx;

						.yqbox-cp-title {
							padding: 30rpx 20rpx 17rpx;
							font-size: 30rpx;
							color: #ffffff;
							overflow: hidden;
							text-overflow: ellipsis;
							white-space: nowrap;
						}

						.yqbox-cp-text {
							font-size: 20rpx;
							font-family: Microsoft YaHei;
							color: #EEE6B6;
						}
					}

					.yqboxCp-right {
						width: calc(100% - 300rpx);
						padding-top: 20rpx;
						padding-left: 66rpx;
					}
				}
			}

			.yqbox-btn {
				margin-top: 30rpx;
				width: 562rpx;
				height: 80rpx;
				background: #F1E2BC;
				border-radius: 29rpx;
				font-size: 30rpx;
				font-family: Microsoft YaHei;
				font-weight: bold;
				color: #000;
				text-align: center;
				line-height: 80rpx;
			}
		}
	}

	.yuyue {
		background-color: #FFFFFF;
		padding: 20rpx;
		margin: 0 30rpx;
		margin-top: 30rpx;
		border-radius: 10rpx;
		box-shadow: 0rpx 0rpx 15rpx 6rpx rgba(52, 52, 52, 0.1);

		.yuyue-title {
			font-size: 30rpx;
			font-family: Microsoft YaHei;
			font-weight: bold;
			color: #000000;
			padding-bottom: 20rpx;
			border-bottom: 1rpx solid #000000;
		}

		.yuyue-name {
			width: 100%;
			margin-top: 30rpx;
			display: flex;
			justify-content: space-between;

			.yuyue-name-list {
				font-size: 24rpx;
				font-weight: 600;
				color: #000;
			}
		}

		.yuyue-con {
			width: 100%;
			height: 30rpx;
			margin-top: 30rpx;
			display: flex;
			justify-content: space-between;
			position: relative;

			.yuyue-con-list {
				width: 30rpx;
				height: 30rpx;
				background-color: #f1e2bc;
				border-radius: 50%;
				z-index: 100;
			}
		}

		.yuyue-con::after {
			content: "";
			display: block;
			width: 100%;
			height: 2rpx;
			background-color: #000;
			position: absolute;
			top: 0;
			bottom: 0;
			margin-top: auto;
			margin-bottom: auto;
		}

		.yuyue-time {
			margin-top: 20rpx;
			width: 100%;
			display: flex;
			justify-content: space-between;

			.yuyue-time-list {
				width: 86rpx;
				min-height: 60rpx;
				font-size: 24rpx;
				line-height: 30rpx;
			}
		}
	}

	.yq {
		background-color: #FFFFFF;
		padding: 20rpx;
		margin: 0 30rpx;
		margin-top: 30rpx;
		border-radius: 10rpx;
		box-shadow: 0rpx 0rpx 15rpx 6rpx rgba(52, 52, 52, 0.1);

		.yq-top {
			width: 100%;
			display: flex;
			justify-content: space-between;

			.yq-top-left {
				font-size: 30rpx;
				font-family: Microsoft YaHei;
				font-weight: bold;
				color: #000000;
			}

			.yq-top-right {
				display: flex;
				align-items: center;
				font-size: 24rpx;
				font-family: Microsoft YaHei;
				font-weight: bold;
				color: #999999;
			}
		}

		.yq-lu {
			margin: 20rpx 0;
			width: 221rpx;
			height: 51rpx;
			background: #D1F1C2;
			border-radius: 26rpx;
			font-size: 24rpx;
			color: #999999;
			text-align: center;
			line-height: 51rpx;
		}

		.img {
			width: 100%;
			height: 58rpx;
			background: #252525;
			border-radius: 50rpx;
			display: flex;
			justify-content: space-between;

			.img-list {
				width: 58rpx;
				height: 58rpx;
				border-radius: 50%;
			}
		}

		.yq-con {
			margin-top: 30rpx;
			width: 100%;
			height: 30rpx;
			display: flex;
			justify-content: space-between;

			.yq-con-text {
				font-size: 26rpx;
				font-family: Microsoft YaHei;
				font-weight: bold;
				color: #999999;
			}

			.yq-active {
				color: #000;
			}
		}
	}

	.pop {
		width: 548rpx;
		background: #2D2F2E;
		border-radius: 10rpx;
		padding: 20rpx;

		.pop-top {
			width: 100%;
			height: 36rpx;
			display: flex;
			align-items: center;
			justify-content: center;
			margin-top: 40rpx;
			font-size: 36rpx;
			font-family: Microsoft YaHei;
			font-weight: bold;
			color: #FFFFFF;

			.pop-top-img {
				width: 36rpx;
				height: 36rpx;
				margin-right: 40rpx;
			}
		}

		.pop-con {
			margin-top: 40rpx;
			font-size: 28rpx;
			font-family: Microsoft YaHei;
			font-weight: bold;
			color: #FFFFFF;
			text-align: center;
			line-height: 60rpx;
		}

		.pop-btn {
			margin: 30rpx auto;
			width: 382rpx;
			height: 74rpx;
			background: #F1E2BC;
			border-radius: 37rpx;
			font-size: 30rpx;
			font-weight: bold;
			color: #000000;
			text-align: center;
			line-height: 74rpx;
		}
	}

	.subBtn1 {
		width: 360rpx;
		height: 88rpx;
		// background: url(../../static/img/index/button.png) no-repeat;
		background: #F1E2BC;
		border-radius: 44rpx;
		margin-right: 30rpx;

		text {
			margin-top: 4rpx;
			display: block;
			text-align: center;
			font-family: PingFangSC-Medium, PingFang SC;

		}

		text:first-child {
			font-size: 30rpx;
			color: #000;
			font-weight: 600;
		}

		text:last-child {
			font-size: 24rpx;
			color: #999999;
		}
	}

	.content {
		padding-bottom: 150rpx;

		.bigbox {
			// background: url(../../static/img/detail-bg.png) no-repeat left top;
			background: url(../../static/img/index/bg.jpg) no-repeat left top;
			background-size: 100%;
			background-color: #000;
		}

		.Box {
			width: 100%;
			height: 680rpx;
			padding-bottom: 50rpx;
			background: url(../../static/img/index/bj1.png) no-repeat left top;
			background-size: 100%;
			display: flex;
			flex-direction: column;
			justify-content: flex-end;
			align-items: center;
			perspective: 1000;
			-webkit-perspective: 1000;

			.videoBox {
				width: 630rpx;
				height: 473rpx;

				video {
					width: 100%;
					height: 100%;
				}
			}

			.rotateBox {
				width: 540rpx;
				height: 540rpx;
				animation: myfirst 10s linear 150ms 100;
				z-index: 200;

				.image {
					width: 540rpx;
					height: 540rpx;

				}
			}
		}

		.Box1 {
			width: 100%;
			height: 360rpx;
			margin-top: -120rpx;
			padding-bottom: 30rpx;
			background: url(../../static/img/index/bj2.png) no-repeat top center;
			background-size: 620rpx;
			display: flex;
			justify-content: center;
			align-items: flex-end;

			.img {
				width: 84rpx;
				height: 152rpx;
				margin: 0 15rpx;
			}

			.center {
				max-width: 500rpx;
				height: 152rpx;
				display: flex;
				flex-direction: column;
				justify-content: space-around;
				align-items: center;

				.goodsName {
					color: #FFFFFF;
					font-size: 30rpx;
					text-align: center;
					overflow: hidden;
					word-break: break-all;
					text-overflow: ellipsis;
					display: -webkit-box;
					-webkit-box-orient: vertical;
					-webkit-line-clamp: 2;
				}

				.LimitBox {
					// width: 200rpx;
					height: 40rpx;
					line-height: 40rpx;
					background: #4C464A;
					border-radius: 6rpx;
					font-size: 24rpx;
					text-align: center;

					.Limit {
						min-width: 60rpx;
						padding: 0 10rpx;
						border-radius: 6rpx;
						color: #010101;
						background-color: #F1E2BC;
					}

					.stock {
						min-width: 80rpx;
						padding: 0 10rpx;
						color: #F1E2BC;
					}
				}
			}
		}

		.msgBox {
			background-color: #FFFFFF;
			// background-color: #23272C;
			padding: 20rpx;
			margin: 0 30rpx;
			border-radius: 10rpx;
			box-shadow: 0rpx 0rpx 15rpx 6rpx rgba(52, 52, 52, 0.1);

			.goodsName {
				// color: #333333;
				color: #FFFFFF;
				font-size: 36rpx;
				font-weight: 500;
				margin-bottom: 20rpx;
			}

			.category_name {
				height: 40rpx;
				line-height: 40rpx;
				font-size: 22rpx;
				padding: 0 15rpx;
				background: #F1E2BC;
				border-radius: 6rpx;
				color: #1E1817;
				margin-right: 20rpx;
			}

			.number {
				height: 40rpx;
				line-height: 40rpx;
				padding: 0 15rpx;
				border-radius: 20rpx;
				background-color: #FFDD9D;
				color: #8A683A;
				font-size: 20rpx;
				font-weight: 500;
			}

			.describe {
				color: #666666;
				font-size: 26rpx;
				line-height: 38rpx;
				font-weight: 500;
				margin-top: 20rpx;
			}

			.priceBox {
				// color: #AE3523;
				color: #000000;
				font-size: 24rpx;
				font-weight: 500;

				text {
					font-weight: 500;
					font-size: 36rpx;
					margin-left: 10rpx;
				}
			}

			.rule {
				// background: #F4F4F4;
				background: #ffffff;
				border-radius: 20rpx;
				padding: 20rpx;
				color: #777777;
				font-size: 24rpx;
				line-height: 34rpx;
				margin-top: 20rpx;
				box-shadow: 0px 0px 15rpx 6rpx rgba(52, 52, 52, 0.1);
				border-radius: 10rpx;

				.rule1 {
					color: #000000;
					margin-bottom: 10rpx;
				}
			}
		}

		.descBox {
			margin: 0 30rpx;
			padding: 20rpx 30rpx;
			background-color: #ffffff;
			color: #AAAAAA;
			box-shadow: 0px 0px 15rpx 6rpx rgba(52, 52, 52, 0.1);
			border-radius: 10rpx;

			.item {
				font-size: 26rpx;
				font-weight: 500;
				color: #000000;
				margin-bottom: 10rpx;
			}
		}

		.type1 .goodsinfo {
			padding: 0 30rpx;
			background-color: #FFFFFF;
			// background-color: #23272C;
			margin: 20rpx 30rpx;
			box-shadow: 0px 0px 15rpx 6rpx rgba(52, 52, 52, 0.1);
			border-radius: 10rpx;

			.iptBox {
				padding: 20rpx 0;
				border-bottom: 1rpx solid #F0F0F0;

				.label {
					width: 20%;
					font-size: 24rpx;
					color: 000000;
				}

				.center {
					width: 75%;
					color: #AAAAAA;
					font-size: 24rpx;
					min-height: 40rpx;
					word-break: break-word;
					display: block;
					line-height: 44rpx;
					position: relative;
				}

				.copy {
					width: 44rpx;
					height: 44rpx;
					position: absolute;
					bottom: 0;
					right: 0;
				}
			}
		}

		.footerBox {
			position: fixed;
			left: 0;
			bottom: 0;
			z-index: 100;
			width: 100%;
			height: 120rpx;
			background-color: #FFFFFF;
			// border-top: 1rpx solid #BFBFBF;
			// background-color: rgba(0, 0, 0, 0.7);
			box-shadow: 0rpx -4rpx 32rpx 0rpx rgba(180, 180, 180, 0.5);

			.price {
				// color: #AE3523;
				color: #000000;
				font-size: 40rpx;
				margin-left: 30rpx;
			}

			.count {
				color: #777777;
				font-size: 28rpx;
				margin-left: 30rpx;
			}

			.subBtn {
				width: 360rpx;
				height: 88rpx;
				line-height: 88rpx;
				text-align: center;
				font-size: 30rpx;
				font-family: PingFangSC-Medium, PingFang SC;
				color: #333333;
				font-weight: 500;
				// background: url(../../static/img/index/button.png) no-repeat;
				background: #00DB7D;
				border-radius: 44rpx;
				margin-right: 30rpx;
			}


			.subnrn1 {
				background: #8c8c8c;
			}
		}

		.tech-tip {
			margin-top: 30rpx;
			display: flex;
			justify-content: center;

			image {
				width: 427rpx;
				height: 42rpx;
			}
		}

	}
</style>
