<template>
	<view class="content container">
		<view class="status_bar flexBox" :class="[scrollTopt>0?'fixedbotto':'','']">
			<view class="search">
				<input type="text" v-model="search" placeholder="请输入搜索内容">
				<view class="search-btn" @tap="searchClick">
					搜索
				</view>
			</view>
		</view>
		<swiper class="banner" indicator-dots="true" circular="true" autoplay="true" interval="3000" duration="800"
			indicator-color="#FFFFFF" indicator-active-color="#00D18B">
			<!-- next-margin="190rpx" -->
			<swiper-item v-for="(item, index) in banner" :key="index" @tap="next1(item)">
				<image :src="item.image" mode=""></image>
			</swiper-item>
		</swiper>
		<view class="notice">
			<u-notice-bar :text="text1" bgColor="#ffffff"></u-notice-bar>
		</view>
		<!-- 三个筛选 -->
		<!-- <view :class="[scrollTopt>230?'fixedbotto':'','tabBox']">
			<view @tap="tab('0')" :class="showType == '0' ? 'tab act' : 'tab'">
				<text>数字藏品</text>
				<view class="line"></view>
			</view>
			<view @tap="tab('1')" :class="showType == '1' ? 'tab act' : 'tab'">
				<text>发售日历</text>
				<view class="line"></view>
			</view>
			<view @tap="tab('2')" :class="showType == '2' ? 'tab act' : 'tab'">
				<text>内容精选</text>
				<view class="line"></view>
			</view>
		</view> -->
		<view class="listBox" v-if="showType=='0'">
			<view class="listItem" v-for="(item,index) in shopList" :key="index" @tap="next(item)">
				<view class="mask">
					<view class="state" v-if="item.status == 1 &&  item.surplus > 0">{{item.end_time}} 结束</view>
					<view class="state state1" v-if="item.status == 1 &&   (item.surplus < 0 || item.surplus == 0)">
						<image src="../../static/img/index/time.png" mode="" class="time"></image>
						已售罄
					</view>
					<view class="state" v-if="item.status == 2 ">{{item.start_time}} 开售</view>
					<view class="state state1" v-if="item.status == 3 && (item.surplus < 0 || item.surplus == 0)">
						<image src="../../static/img/index/time.png" mode="" class="time"></image>
						已售罄
					</view>
					<view class="state state1" v-if="item.status == 3 && item.surplus > 0 ">
						<image src="../../static/img/index/time.png" mode="" class="time"></image>
						已结束
					</view>
					<view class="center flex_column">
						<image :src="item.image" mode="aspectFill"></image>
					</view>
				</view>
				<image class="goodsImg" :src="item.image" mode=""></image>
				<view class="goodsinfo">
					<view class="goodsinfo-box">
						<view class="goodName">{{item.name}}</view>
						<view class="price">¥{{item.price}}</view>
					</view>
					<view class="flexBox" style="margin-top: 10rpx;">
						<view class="label">{{item.goods_category_name}}</view>
						<view class="flexBox LimitBox">
							<view class="Limit">限量</view>
							<view class="stock">{{item.surplus}}</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="plr-30" v-if="showType=='1'">
			<view class="con bg1 ptb-24 plr-24">
				<view v-for="(item,index) in shopList" :key="index">
					<view class="title flex">
						<image src="../../static/img/data.png" mode="" class="width36 height34"></image>
						<view class="size-26 black ml-12">{{item.time}}</view>
					</view>
					<view v-for="(itm,idex) in item.data" :key="idex">
						<view class="size-24 black mt-16 mb-20">{{itm.time}}</view>
						<view v-for="(itm2,idx) in itm.data" :key="idx">
							<view class="conlist mb-30 flex" v-for="(itm3,idx3) in itm2.goods" :key="idx3">
								<image :src="itm3.image" mode="aspectFill" class="conimg"></image>
								<view class="conright ptb-20">
									<view class="black size-28 mb-12">{{itm3.name}}</view>
									<view class="flex">
										<view class="label">{{itm3.goods_category_name}}</view>
										<view class="labelwrapper">
											<view class="label ">限量</view>
											<text class="ml-16">{{itm3.stock}}份</text>
										</view>
									</view>
									<view class="size-36 black mt-16">
										<text class="size-26">¥</text>
										{{itm3.price}}
									</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="jxbox" v-if="showType=='2'">
			<view class="list mt-20" v-for="(item,index) in shopList" :key="index" @click="detail(item)">
				<image :src="item.image" mode="" class="twoimg"></image>
				<view class="size-30 black bold plr-30 mt-10">{{item.title}}</view>
				<view class="jxbox-content size-26 mt-10 plr-30" style="color: #666666;word-break:break-all;">
					<!-- <u-parse :content="item.content"></u-parse>-->
					{{item.desc}}
				</view>
			</view>
		</view>
		<uni-load-more :status="status" v-if="shopList.length"></uni-load-more>
		<uni-footer currentTab="0"></uni-footer>
	</view>
</template>
<script>
	export default {
		data() {
			return {
				search: "",
				banner: [],
				shopList: [],
				status: 'more',
				pagesize: 15,
				page: 1,
				flag: false, //上拉加载\
				block: false,
				gid: '0',
				showType: '0',
				scrollTopt: 0,
				text1: "uView UI众多组件覆盖开发过程的各个需求，组件功能丰富，多端兼容。让您快速集成，开箱即用"
			}
		},
		onLoad(e) {
			this.getBanner()
			this.shopList = [];
			this.getList();
			const that = this
			uni.$on('init', () => {
				if (that.showType == '0') {
					that.flag = false;
					that.page = 1;
					that.shopList = [];
					that.getList();
				}
			})
			this.getNotice();
		},
		onPageScroll(e) {
			this.scrollTopt = e.scrollTop;
			if (this.scrollTopt > 220) {}
		},
		onReachBottom() {
			if (!this.flag) {
				this.status = "loading";
				this.page++;
				this.getList();
			}
		},
		onPullDownRefresh() {
			this.getBanner();
			this.flag = false;
			this.page = 1;
			this.shopList = [];
			this.getList();
		},
		methods: {
			async getNotice() {
				try {
					let res = await this.$http.get("index/noticeList");
					this.text1 = res.data[0].content;
				} catch (e) {

				}
			},
			next1(a) {
				if (a.url) {
					// #ifdef MP || APP-PLUS
					this.go('webView?ss=' + a.url)
					// #endif
					// #ifdef H5
					location.href = a.url;
					// #endif
				} else {
					this.go('secondHand')
				}
			},
			tab(n) {
				if (this.showType != n) {
					this.showType = n;
					this.flag = false;
					this.page = 1;
					this.shopList = [];
					this.getList();
				}
			},
			searchClick() {
				this.flag = false;
				this.page = 1;
				this.shopList = [];
				this.getList();
			},
			reload(a) {
				if (a == '0') {
					this.block = !this.block
				}
				if (this.gid != a) {
					this.gid = a;
					this.shopList = [];
					this.getList();
				}

			},
			detail(con) {
				this.go('txtdetail?con=' + JSON.stringify(con))
			},
			next(a) {
				var text = ''
				var flag = 'false'
				if (a.status == 1 && (a.surplus < 0 || a.surplus == 0)) {
					text = '已售罄'
					flag = 'true'
				}
				if (a.status == 2) {
					text = '未开始'
					flag = 'true'
				}
				if (a.status == 3 && (a.surplus < 0 || a.surplus == 0)) {
					text = '已售罄'
					flag = 'true'
				}
				if (a.status == 3 && a.surplus > 0) {
					text = '已结束'
					flag = 'true'
				}
				this.go('goodsDetail?goodsId=' + a.id + '&flag=' + flag + '&text=' + text + '&status=' + a.status)
			},
			getList() {
				/* if (this.showType != '0') {
					this.$http.get(this.showType == '2' ? 'index/noticeList' : 'index/calendar', {
						page: this.page,
						pagesize: this.pagesize,
						search: this.search
					}).then(res => {
						if (res.code == 1) {
							if (res.data.length == 0) {
								this.flag = true;
								this.status = 'noMore'
							} else {
								this.shopList = this.shopList.concat(res.data);
								if (res.data.length < this.pagesize) {
									this.flag = true;
									this.status = 'noMore'
								}
							}
						} else {
							this.flag = true;
							this.status = 'noMore'
						}
					})
					return false
				} */
				this.$http.get('goods/goodsList', {
					goods_category_id: this.gid,
					page: this.page,
					pagesize: this.pagesize,
					search: this.search
				}).then(res => {
					uni.stopPullDownRefresh();
					if (res.code == 1) {
						if (res.data.data.length == 0) {
							this.flag = true;
							this.status = 'noMore'
						} else {
							this.shopList = this.shopList.concat(res.data.data);
							if (res.data.data.length < this.pagesize) {
								this.flag = true;
								this.status = 'noMore'
							}
						}
					} else {
						this.flag = true;
						this.status = 'noMore'
					}
				})
			},
			getBanner() {
				this.$http.get('index/bannerList', {
					type: '1'
				}).then(res => {
					uni.stopPullDownRefresh();
					if (res.code == 1) {
						this.banner = res.data;
					}
				})
			}
		}
	}
</script>
<style lang="scss" scoped>
	/deep/uni-swiper .uni-swiper-wrapper {
		overflow: inherit;
	}

	/deep/uni-swiper .uni-swiper-dots-horizontal {
		bottom: -20rpx !important;
	}

	/deep/uni-swiper .uni-swiper-dot-active {
		width: 30rpx;
		height: 12rpx;
		background: #00D18B;
		border-radius: 6rpx;
	}

	.notice {
		margin: 0 30rpx;
		padding: 0 20rpx;
		background: #FFFFFF;
		box-shadow: 0px 0px 15rpx 6rpx rgba(52, 52, 52, 0.1);
		border-radius: 20rpx;
	}

	.content {
		padding-bottom: 100rpx;

		.twoimg {
			height: 400rpx;
			width: 100%;
			border-radius: 20rpx;
		}

		.status_bar {
			width: calc(100% - 40rpx);
			padding: 0 20rpx;
			height: 80rpx;

			padding-top: var(--status-bar-height);

			.search {
				width: calc(100%);
				height: 60rpx;
				border-radius: 10rpx;
				box-shadow: 0rpx 0rpx 15rpx 6rpx rgba(52, 52, 52, 0.1);
				display: flex;

				input {
					width: calc(100% - 140rpx);
					;
					height: 60rpx;
					font-size: 26rpx;
					padding: 0 20rpx;
					border-radius: 5rpx;
				}

				.search-btn {
					width: 99rpx;
					height: 60rpx;
					border-left: 1rpx solid rgba(52, 52, 52, 0.1);
					font-size: 28rpx;
					color: #000000;
					text-align: center;
					line-height: 60rpx;
				}
			}
		}

		.banner {
			padding: 0 0 20rpx 0;
			// padding-top: calc(var(--status-bar-height) + 130rpx);
			width: calc(100%);
			height: 320rpx;
			overflow: hidden;

			.uni-swiper-wrapper {
				z-index: 10;
				// background: #0A0A0A;
			}

			image {
				width: 100%;
				height: 320rpx;
			}

			.info {
				padding: 20rpx;

				.name {
					color: #FFFFFF;
					font-size: 30rpx;
				}

				.title {
					color: #757679;
					font-size: 22rpx;
				}
			}
		}

		.tabBox {
			width: 100%;
			// padding-top: 12rpx;
			display: flex;

			.tab {
				width: 33%;
				position: relative;
				font-size: 30rpx;
				color: #666666;
				height: 90rpx;
				line-height: 90rpx;
				text-align: center;

				.line {
					position: absolute;
					left: calc(50% - 25rpx);
					bottom: 0rpx;
					width: 40rpx;
					height: 16rpx;
					background-image: url(../../static/img/line.png);
					background-repeat: no-repeat;
					background-size: 40rpx 16rpx;
					display: none;
				}

				&.act {
					font-size: 30rpx;
					font-weight: bold;
					color: #000000;

					.line {
						display: block;
					}
				}
			}
		}

		.fixedbotto {
			display: flex;
			position: fixed;
			top: 0;
			background: #FFFFFF;
			z-index: 99;
		}

		.listBox {
			margin: 20rpx 20rpx;
			display: flex;
			flex-wrap: wrap;

			.listItem {
				width: 345rpx;
				background-color: #FFFFFF;
				border-radius: 30rpx;
				margin-bottom: 35rpx;
				margin-right: 20rpx;
				position: relative;

				.mask {
					width: 345rpx;
					height: auto;
					border-radius: 30rpx;
					position: absolute;
					left: 0;
					top: 0;
					z-index: 10;

					.state {
						display: inline-block;
						padding: 0 18rpx;
						height: 40rpx;
						line-height: 40rpx;
						text-align: center;
						color: #fff;
						font-size: 24rpx;
						border-radius: 22rpx;
						background: linear-gradient(90deg, #00D18B 0%, #00D18B 100%);
						margin-top: 30rpx;
						margin-left: 28rpx;
					}

					.state1 {
						color: rgba(255, 255, 255, 0.6);
						background: #010004;
						border-radius: 22rpx;
						font-size: 24rpx;
						font-family: PingFangSC-Regular, PingFang SC;
						font-weight: 400;
						color: #FFFFFF;
						display: flex;
						max-width: 150rpx;
						box-sizing: border-box;

						.time {
							height: 26rpx;
							width: 26rpx;
							margin-right: 8rpx;
							margin: auto 8rpx auto 0;
						}
					}

					.center {
						width: 235rpx;
						height: 177rpx;
						background: url(../../static/img/index/bj.png) no-repeat left top;
						background-size: 100%;
						margin: 40rpx auto;

						image {
							width: 150rpx;
							height: 150rpx;
							border-radius: 12rpx;
						}
					}
				}

				.goodsImg {
					width: 345rpx;
					height: 345rpx;
					border-radius: 30rpx;
					-webkit-filter: blur(4px);
					filter: blur(4px);
				}

				.goodsinfo {
					padding: 15rpx 20rpx;

					.goodsinfo-box {
						width: 100%;
						display: flex;
						justify-content: space-between;
					}

					.goodName {
						color: #000000;
						font-size: 30rpx;
						margin-bottom: 15rpx;
					}

					.label {
						height: 40rpx;
						line-height: 40rpx;
						font-size: 22rpx;
						padding: 0 10rpx;
						background: #F1E2BC;
						border-radius: 6rpx;
						color: #1E1817;
						margin-right: 10rpx;
					}

					.LimitBox {
						height: 40rpx;
						line-height: 40rpx;
						background: #3A3A3A;
						border-radius: 6rpx;
						font-size: 12px;

						.Limit {
							font-size: 24rpx;
							padding: 0 10rpx;
							border-radius: 6rpx;
							color: #010101;
							background-color: #EDDFC0;
						}

						.stock {
							font-size: 24rpx;
							padding: 0 15rpx;
							color: #fff;
						}
					}

					.bottom {
						margin-top: 10rpx;
					}

					.company_image {
						width: 32rpx;
						height: 32rpx;
						margin-right: 10rpx;
					}

					.price {
						color: #000000;
						font-size: 30rpx;
					}

					.good {
						flex: 1;
						color: rgba(255, 255, 255, 0.35);
						// color: #777777;
						font-size: 24rpx;
						font-weight: 500;
					}
				}
			}

			.listItem:nth-child(2n) {
				margin-right: 0rpx;
			}
		}

		.conlist {
			height: 200rpx;
			// background: url(../../static/img/index/bg1.png) no-repeat;
			// background-size: 100% auto;
			background: #F0F0F0;
			border-radius: 30rpx;

			.conimg {
				width: 200rpx;
				height: 200rpx;
				border-radius: 30rpx;
			}

			.conright {
				margin-left: 24rpx;
				flex: 1;

				.label {
					height: 40rpx;
					line-height: 40rpx;
					background: #EDDFC0;
					border-radius: 6rpx;
					font-size: 22rpx;
					padding: 0 16rpx;
					font-family: PingFangSC-Regular, PingFang SC;
					font-weight: 400;
					color: #333333;
				}

				.labelwrapper {
					height: 40rpx;
					line-height: 40rpx;
					background: #59699A;
					border-radius: 6rpx;
					font-size: 22rpx;
					font-family: PingFangSC-Regular, PingFang SC;
					font-weight: 400;
					color: #FFFFFF;
					padding-right: 16rpx;
					margin: 0 16rpx;
					display: flex;
				}
			}
		}

		.jxbox {
			width: 100%;
			padding: 0 30rpx;

			.jxbox-content {
				width: calc(100% - 60rpx);
				word-wrap: break-word;
			}
		}

		.list {
			width: calc(100% - 60rpx);
			background: #F0F0F0;
			border-radius: 20rpx;
			padding-bottom: 20rpx;
		}
	}
</style>
