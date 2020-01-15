<template>
	<view class="productDetail">
		<!-- 轮播图 -->
		<Swiper-List :bannerList="product.banner" :height="750"></Swiper-List>
		<!-- 价格标题 -->
		<view class="title-wrapper">
			<view class="price-content">
				<text class="sign">¥</text>
				<text class="price">{{ product.price }}</text>
			</view>
			<view class="name">{{ product.title }}</view>
		</view>
		<!-- 促销 -->
		<view class="discount">
			<text class="title">促销</text>
			<text class="content">{{ product.discounts }}</text>
		</view>
		<!-- 规格 -->
		<view class="guige-wrapper">
			<view class="item-wrapper" @tap="toggleSpec">
				<text class="title">已选</text>
				<text class="content">{{ selectArr.length > 0 ? selectArr.join(' ') : '请选择规格' }}</text>
				<view class="right-icon"><text class="icons">></text></view>
			</view>
			<view class="item-wrapper">
				<text class="title">运费</text>
				<text class="content">{{ product.freight }}</text>
			</view>
		</view>
		<!-- 商品评价只展示5个 -->
		<view class="comment-wrapper">
			<view class="comment-title-wrapper">
				<text class="comment-num">商品评价(32)</text>
				<text class="comment-more">查看全部 ></text>
			</view>
			<scroll-view class="comment-content-wrapper" scroll-x="true">
				<view class="comment-item" v-for="(item, index) in product.commonts" :key="index">
					<view class="comment-item-left">
						<view class="head-content-wrapper">
							<view class="head-icon-content">
								<image class="head-icon" lazy-load="true" :src="item.icon"></image>
							</view>
							<view class="head-name">{{ item.nickname }}</view>
						</view>
						<view class="comment-content">{{ item.comment }}</view>
					</view>
					<view class="comment-item-right">
						<image class="product-img" lazy-load="true" :src="item.productImg"></image>
					</view>
				</view>
			</scroll-view>
		</view>
		<!--商品详情  -->
		<view class="detail-wrapper">
			<view class="title">商品详情</view>
			<view class="content">
				<image class="product-detail-img" mode="widthFix" lazy-load="true" :src="product.details"></image>
			</view>
		</view>
		<!-- 底部footer -->
		<view class="footer-wrapper">
			<view class="footer-icon-wrapper">
				<view class="footer-icon-item">
					<view class="footer-icon"></view>
					<text class="footer-text">首页</text>
				</view>
				<view class="footer-icon-item">
					<view class="footer-icon"></view>
					<text class="footer-text">客服</text>
				</view>
				<view class="footer-icon-item">
					<view class="footer-icon"></view>
					<text class="footer-text">购物车</text>
				</view>
			</view>
			<view class="footer-btn-wrapper">
				<button class="footer-btn add-cart">加入购物车</button>
				<button class="footer-btn buy">立即购买</button>
			</view>
		</view>

		<!-- sku弹出层 -->
		<view class="popup" catchtouchmove="true" :class="specClass" @touchmove.stop.prevent="moveHandle">
			<view class="mask"></view>
			<view class="layer attr-content">
				<view class="specification-wrapper">
					<scroll-view class="specification-wrapper-content" scroll-y="true">
						<view class="specification-header">
							<view class="specification-left">
								<image class="product-img"></image>
							</view>
							<view class="specification-right">
								<view class="price-content">
									<text class="sign">¥</text>
									<text class="price">{{ selectshop.price || 0 }}</text>
								</view>
								<view class="inventory">库存:{{ selectshop.stock || 0 }}</view>
								<view class="choose">已选:{{ selectArr.join(' ') }}</view>
							</view>
						</view>
						<view class="specification-content">
							<view class="specification-item" v-for="(item, index1) in specifications" :key="index1">
								<view class="item-title">{{ item.name }}</view>
								<view class="item-wrapper">
									<view class="item-content" @tap="skuClick(item_value, index1, $event, index2)" v-for="(item_value, index2) in item.item"
									 :key="index2" :class="[item_value.ishow==false ? 'noactived' : '', subIndex[index1] == index2 ? 'actived' : '']">
										{{ item_value.name }}
									</view>
								</view>
							</view>
							<view class="specification-item">
								<view class="item-title">数量</view>
								<view class="item-wrapper">
									<stepper size="small" :min="1" :max="selectshop.stock" :defaultValue="selectNum" :display="canCount" @change="changeNum"></stepper>
								</view>
							</view>
						</view>
					</scroll-view>
					<view class="close" @tap="closeSf">
						<image class="close-item" src="../../static/close.png"></image>
					</view>
				</view>
				<view class="btn-wrapper"><button class="sure" @click="getGoodsSpec">确定</button></view>
			</view>
		</view>
	</view>
</template>

<script>
	import SwiperList from '@/components/other/swiperList.vue';
	import stepper from '@/components/other/stepper.vue';
	import products from '@/json/product.json';
	import skuList from '@/json/skuList.json';
	import specList from '@/json/specList.json';

	export default {
		components: {
			SwiperList,
			stepper
		},
		data() {
			return {
				product: products.product,
				showSet: false,
				specClass: 'none',
				specifications: specList.specifications, //spu规格列表
				difference: skuList.differenceList, //sku列表 {"id": "23","price": 500,"stock": 48,"difference": ["100","绿色","X","豪华"]}
				shopItemInfo: {}, //存放要和选中的值进行匹配的数据
				selectArr: [], //存放被选中的值
				subIndex: [], //是否选中 因为不确定是多规格还是但规格，所以这里定义数组来判断
				selectshop: {}, //存放最后选中的商品
				selectNum: 1 //选中数量
			};
		},
		computed: {
			canCount() {
				return this.subIndex.some(item => item === -1);
			}
		},
		onLoad() {



			console.log(products);
			console.log(skuList.differenceList, 'skuList-比对数据');
			console.log(specList.specifications, 'specList-渲染数据');

			this.specifications.map(item => {
				this.selectArr.push('');
				this.subIndex.push(-1);
			});
			this.changeData(this.difference)
			this.checkItem(); //计算sku里面规格形成路径
			this.checkInpath(-1); //传-1是为了不跳过循环
			console.log(this.selectArr,this.specifications);
		},
		methods: {
			getGoodsSpec(){
				// this.selectshop  
				
				console.log( this.selectshop  ,this.selectNum )
			},

			changeData(differenceList) {
				for (let item of differenceList) {
				let nowOneCompound = []
					for (let oneCompound of item.compound) {				
						nowOneCompound.push(oneCompound.value)
					}
					item.difference = nowOneCompound
				}
				console.log(differenceList, 'difference--changdata')
				
			},


			skuClick(value, index1, event, index2) {
				if (value.ishow) {
					if (this.selectArr[index1] != value.name) {
						this.$set(this.selectArr, index1, value.name);
						this.$set(this.subIndex, index1, index2);
					} else {
						this.$set(this.selectArr, index1, '');
						this.$set(this.subIndex, index1, -1);
					}
					this.checkInpath(index1);
					//如果全部选完
					if (this.selectArr.every(item => item != '')) {
						this.selectshop = this.shopItemInfo[this.selectArr];
						this.selectNum = 1;
					}
				}
			},
			checkInpath(clickIndex) {
				// console.time('筛选可选路径需要的时间是');
				//循环所有属性判断哪些属性可选
				//当前选中的兄弟节点和已选中属性不需要循环
			
				for (let i = 0, len = this.specifications.length; i < len; i++) {
					if (i == clickIndex) {
						continue;
					}
					let len2 = this.specifications[i].item.length;
					for (let j = 0; j < len2; j++) {
						if (this.subIndex[i] != -1 && j == this.subIndex[i]) {
							continue;
						}
						let choosed_copy = [...this.selectArr];
						this.$set(choosed_copy, i, this.specifications[i].item[j].name);
						console.log(choosed_copy,'choosed_copy')
						let choosed_copy2 = choosed_copy.filter(item => item !== '' && typeof item !== 'undefined');
						console.log(choosed_copy2,this.shopItemInfo,'choosed_copy2')
						if (this.shopItemInfo.hasOwnProperty(choosed_copy2)) {
							this.$set(this.specifications[i].item[j], 'ishow', true);
						} else {
							this.$set(this.specifications[i].item[j], 'ishow', false);
						}
					}
				}
				// console.log(this.specifications)
				// console.timeEnd('筛选可选路径需要的时间是');
			},
			checkItem() {
				// console.time('计算有多小种可选路径需要的时间是');
				//计算有多小种可选路径
				let result = this.difference.reduce(
					(arrs, items) => {
						return arrs.concat(
							items.difference.reduce(
								(arr, item) => {
									return arr.concat(
										arr.map(item2 => {
											//利用对象属性的唯一性实现二维数组去重
											if (!this.shopItemInfo.hasOwnProperty([...item2, item])) {
												this.shopItemInfo[[...item2, item]] = items;
											}
											return [...item2, item];
										})
									);
								},
								[[]]
							)
						);
					},
					[[]]
				);
				// console.timeEnd('计算有多小种可选路径需要的时间是');
			},
			
			//规格弹窗开关
			toggleSpec() {
				this.specClass = 'show';
			},
			moveHandle() {
				//禁止父元素滑动
			},
			closeSf() {
				this.specClass = 'hide';
				setTimeout(() => {
					this.specClass = 'none';
				}, 250);
			},
			changeNum(val) {
				this.selectNum = parseInt(val);
			}
		}
	};
</script>

<style lang="less">
	page {
		background-color: #f4f4f4;
	}

	.productDetail {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		z-index: 666;
		overflow: scroll;
		padding-bottom: 120rpx;

		.title-wrapper {
			padding: 40rpx 25rpx;
			background-color: #fff;
			margin-bottom: 20rpx;

			.price-content {
				color: #fe3a3a;
				margin-bottom: 20rpx;

				.sign {
					font-size: 28rpx;
				}

				.price {
					font-size: 48rpx;
					font-weight: 500;
				}
			}

			.name {
				font-size: 32rpx;
				font-weight: bold;
				color: #333333;
				line-height: 40rpx;
			}
		}

		.discount {
			display: flex;
			width: 100%;
			flex-direction: row;
			padding: 35rpx 25rpx;
			font-size: 26rpx;
			box-sizing: border-box;
			background-color: #fff;
			margin-bottom: 20rpx;

			.title {
				font-weight: bold;
				margin-right: 35rpx;
			}

			.content {
				flex: 1;
				line-height: 40rpx;
				font-weight: 500;
				margin-right: 26rpx;
			}
		}

		.guige-wrapper {
			.item-wrapper {
				display: flex;
				width: 100%;
				height: 95rpx;
				align-items: center;
				flex-direction: row;
				padding: 0 25rpx;
				font-size: 26rpx;
				box-sizing: border-box;
				background-color: #fff;
				border-bottom: 1rpx solid #dadada;

				&:last-child {
					border-bottom: 0;
					margin-bottom: 20rpx;
				}

				.title {
					font-weight: bold;
					margin-right: 35rpx;
				}

				.content {
					flex: 1;
					margin-right: 26rpx;
				}

				.right-icon {
					flex: 0 0 14rpx;
					width: 14rpx;
					height: 22rpx;
				}
			}
		}

		.comment-wrapper {
			padding: 36rpx 26rpx 26rpx;
			background-color: #fff;
			box-sizing: border-box;
			margin-bottom: 20rpx;

			.comment-title-wrapper {
				display: flex;
				flex-direction: row;
				justify-content: space-between;
				font-size: 26rpx;
				color: #333333;
				margin-bottom: 33rpx;

				.comment-num {
					font-weight: bold;
				}

				.comment-more {
					font-weight: 500;
				}
			}

			.comment-content-wrapper {
				white-space: nowrap;

				.comment-item {
					display: inline-flex;
					width: 600rpx;
					height: 180rpx;
					flex-direction: row;
					margin-right: 20rpx;
					background-color: #f4f4f4;
					border-radius: 10rpx;
					overflow: hidden;

					:last-child {
						margin-right: 0;
					}

					.comment-item-left {
						flex: 1;
						display: flex;
						height: 100%;
						flex-direction: column;
						padding: 27rpx 31rpx 26rpx 26rpx;
						box-sizing: border-box;
						overflow: hidden;

						.head-content-wrapper {
							display: flex;
							flex-direction: row;
							align-items: center;
							width: 100%;
							height: 43rpx;
							margin-bottom: 27rpx;

							.head-icon-content {
								width: 43rpx;
								height: 43rpx;
								border-radius: 50%;
								overflow: hidden;
								margin-right: 17rpx;

								.head-icon {
									width: 100%;
									height: 100%;
								}
							}

							.head-name {
								flex: 1;
								color: #999999;
								font-size: 24rpx;
								font-weight: 500;
								line-height: 40rpx;
								overflow: hidden;
								text-overflow: ellipsis;
								white-space: nowrap;
							}
						}

						.comment-content {
							width: 100%;
							height: 57rpx;
							line-height: 28rpx;
							font-size: 26rpx;
							font-weight: 500;
							white-space: normal;
							display: -webkit-box;
							-webkit-box-orient: vertical;
							-webkit-line-clamp: 2;
							overflow: hidden;
						}
					}

					.comment-item-right {
						height: 100%;
						width: 180rpx;
						flex: 0 0 180rpx;

						.product-img {
							width: 100%;
							height: 100%;
						}
					}
				}
			}
		}

		.detail-wrapper {
			margin-bottom: 20rpx;
			background-color: #fff;

			.title {
				width: 100%;
				height: 94rpx;
				line-height: 94rpx;
				padding: 0 26rpx;
				font-size: 26rpx;
				font-weight: bold;
				color: #333333;
				box-sizing: border-box;
			}

			.content {
				font-size: 0; //把空白字符去掉
				width: 100%;

				.product-detail-img {
					width: 100%;
					height: auto;
				}
			}
		}

		.footer-wrapper {
			position: fixed;
			bottom: 0;
			left: 0;
			z-index: 666;
			display: flex;
			width: 100%;
			height: 120rpx;
			align-items: center;
			background-color: #fff;
			border-top: 1rpx solid #fbfbfb;
			padding: 0 26rpx;
			box-sizing: border-box;

			.footer-icon-wrapper {
				flex: 1;
				display: flex;
				margin-right: 18rpx;
				justify-content: space-between;

				.footer-icon-item {
					width: 70rpx;
					display: inline-flex;
					flex-direction: column;
					align-items: center;
					justify-content: center;

					.footer-icon {
						width: 40rpx;
						height: 40rpx;
						margin-bottom: 10rpx;
						background-color: #999999;
					}

					.footer-text {
						font-size: 20rpx;
						font-weight: 500;
					}
				}
			}

			.footer-btn-wrapper {
				width: 428rpx;
				height: 76rpx;

				.footer-btn {
					display: inline-block;
					width: 206rpx;
					height: 76rpx;
					border-radius: 38rpx;
					text-align: center;
					line-height: 76rpx;
					color: #fff;
					font-weight: 500;
					font-size: 28rpx;
					margin-right: 16rpx;

					&:last-child {
						margin-right: 0;
					}

					&.add-cart {
						background-color: #ffbe46;
					}

					&.buy {
						background-color: #fe3a3a;
					}
				}
			}
		}

		.set-wrapper {
			position: fixed;
			top: 180rpx;
			right: 26rpx;
			z-index: 3;

			.set-content {
				position: relative;
				width: 234rpx;
				height: 185rpx;
				background-color: #000;
				opacity: 0.7;
				border-radius: 10rpx;

				&::before {
					content: '';
					position: absolute;
					top: -14rpx;
					right: 13rpx;
					width: 0;
					border-width: 0 14rpx 14rpx 14rpx;
					border-style: solid;
					border-color: transparent transparent #000 transparent;
				}

				.set-item {
					display: flex;
					width: 100%;
					height: 92rpx;
					border-bottom: 1rpx solid #dadada;
					align-items: center;
					padding: 0 26rpx;
					box-sizing: border-box;

					&:last-child {
						border-bottom: 0;
					}

					.icon {
						width: 40rpx;
						height: 40rpx;
						margin-right: 14rpx;
						background-color: #ffffff;
					}

					.title {
						font-size: 28rpx;
						color: #ffffff;
						font-weight: 500;
						margin-right: 50rpx;
					}

					.dorp {
						width: 16rpx;
						height: 16rpx;
						background-color: #fd3144;
						border-radius: 50%;
					}
				}
			}
		}

		/*  弹出层 */
		.popup {
			position: fixed;
			left: 0;
			top: 0;
			right: 0;
			bottom: 0;
			z-index: 999;
			overflow: hidden;

			&.show {
				display: block;

				.mask {
					animation: showPopup 0.2s linear both;
				}

				.layer {
					animation: showLayer 0.2s linear both;
				}
			}

			&.hide {
				.mask {
					animation: hidePopup 0.2s linear both;
				}

				.layer {
					animation: hideLayer 0.2s linear both;
				}
			}

			&.none {
				display: none;
			}

			.mask {
				position: fixed;
				top: 0;
				width: 100%;
				height: 100%;
				z-index: 1;
				background-color: rgba(0, 0, 0, 0.3);
			}

			.layer {
				display: flex;
				width: 100%;
				// height: 1014rpx;
				flex-direction: column;
				// min-height: 40vh;
				// max-height: 1014rpx;
				position: fixed;
				z-index: 99;
				bottom: 0;
				border-radius: 10upx 10upx 0 0;
				background-color: #fff;

				.specification-wrapper {
					width: 100%;
					padding: 30rpx 25rpx;
					box-sizing: border-box;

					.specification-wrapper-content {
						width: 100%;
						max-height: 900rpx;
						min-height: 600rpx;

						&::-webkit-scrollbar {
							/*隐藏滚轮*/
							display: none;
						}

						.specification-header {
							width: 100%;
							display: flex;
							flex-direction: row;
							position: relative;
							margin-bottom: 40rpx;

							.specification-left {
								width: 180rpx;
								height: 180rpx;
								flex: 0 0 180rpx;

								.product-img {
									width: 180rpx;
									height: 180rpx;
									background-color: #999999;
								}
							}

							.specification-right {
								flex: 1;
								padding: 0 35rpx 0 28rpx;
								box-sizing: border-box;
								display: flex;
								flex-direction: column;
								justify-content: flex-end;
								font-weight: 500;

								.price-content {
									color: #fe3a3a;
									margin-bottom: 20rpx;

									.sign {
										font-size: 28rpx;
									}

									.price {
										font-size: 48rpx;
									}
								}

								.inventory {
									font-size: 24rpx;
									color: #999999;
									margin-bottom: 14rpx;
								}

								.choose {
									font-size: 28rpx;
									color: #333333;
								}
							}
						}

						.specification-content {
							font-weight: 500;

							.specification-item {
								margin-bottom: 40rpx;

								&:last-child {
									margin-bottom: 0;
								}

								.item-title {
									margin-bottom: 20rpx;
									font-size: 28rpx;
									color: #999999;
								}

								.item-wrapper {
									display: flex;
									flex-direction: row;
									flex-flow: wrap;

									.item-content {
										display: inline-block;
										padding: 15rpx 35rpx;
										border-radius: 10rpx;
										background-color: #ffffff;
										color: #333333;
										font-size: 28rpx;
										margin-right: 20rpx;
										border: 2rpx solid #f4f4f4;
										box-sizing: border-box;

										&.actived {
											border-color: #fe3a3a;
											color: #fe3a3a;
										}

										&.noactived {
											background-color: #f4f4f4;
											border-color: #f4f4f4;
										}
									}
								}
							}
						}
					}

					.close {
						position: absolute;
						top: 30rpx;
						right: 25rpx;
						width: 50rpx;
						height: 50rpx;
						text-align: center;
						line-height: 50rpx;

						.close-item {
							width: 50rpx;
							height: 50rpx;
						}
					}
				}

				.btn-wrapper {
					display: flex;
					width: 100%;
					height: 120rpx;
					flex: 0 0 120rpx;
					align-items: center;
					justify-content: space-between;
					padding: 0 26rpx;
					box-sizing: border-box;

					.layer-btn {
						width: 335rpx;
						height: 76rpx;
						border-radius: 38rpx;
						color: #fff;
						line-height: 76rpx;
						text-align: center;
						font-weight: 500;
						font-size: 28rpx;

						&.add-cart {
							background: #ffbe46;
						}

						&.buy {
							background: #fe3a3a;
						}
					}

					.sure {
						width: 698rpx;
						height: 76rpx;
						border-radius: 38rpx;
						color: #fff;
						line-height: 76rpx;
						text-align: center;
						font-weight: 500;
						font-size: 28rpx;
						background: #fe3a3a;
					}
				}
			}

			@keyframes showPopup {
				0% {
					opacity: 0;
				}

				100% {
					opacity: 1;
				}
			}

			@keyframes hidePopup {
				0% {
					opacity: 1;
				}

				100% {
					opacity: 0;
				}
			}

			@keyframes showLayer {
				0% {
					transform: translateY(120%);
				}

				100% {
					transform: translateY(0%);
				}
			}

			@keyframes hideLayer {
				0% {
					transform: translateY(0);
				}

				100% {
					transform: translateY(120%);
				}
			}
		}
	}
</style>
