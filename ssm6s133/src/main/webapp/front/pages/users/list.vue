<template>
	<mescroll-uni @init="mescrollInit" :up="upOption" :down="downOption" @down="downCallback" @up="upCallback">
		
				        <view class="cu-bar bg-white search" :style="[{top:CustomBar + 'px'}]">
			<view class="search-form round">
				<text class="cuIcon-search"></text>
				<input v-model="searchForm.username" type="text" placeholder="用户名" ></input>
			</view>
			<view class="action">
				<button @tap="search" class="cu-btn bg-gradual-green shadow-blur round">搜索</button>
			</view>
		</view>    
                		        		        
		<view v-if='1 == 1' class="uni-product-list" :style='{"borderRadius":0,"backgroundColor":"rgba(160, 156, 156, 0.82)"}'>
			<view @tap="onDetailTap(product)" class="uni-product" :style='{"borderRadius":"8rpx","backgroundColor":"#fff"}' v-for="(product,index) in list" :key="index">
								                				                				                												<view style="display: flex;justify-content: space-between;">
					<text v-if="isAuth('users','修改')" class="cuIcon-edit" @click.stop="onUpdateTap(product.id)">修改</text>
					<text v-if="isAuth('users','删除')" class="cuIcon-delete" @click.stop="onDeleteTap(product.id)">删除</text>
				</view>
			</view>
		</view>
		
		<view v-if='1 == 3' class="list" :style='{"borderRadius":0,"backgroundColor":"rgba(160, 156, 156, 0.82)"}'>
			<view @tap="onDetailTap(product)" class="listm flex flex-between" :style='{"borderRadius":"8rpx","backgroundColor":"#fff"}' v-for="(product,index) in list" :key="index">
																																																																			</view>
		</view>
		
		<button v-if="isAuth('users','新增')" class="add-btn" @click="onAddTap()">新增</button>
		
	</mescroll-uni>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
				mescroll: null, //mescroll实例对象
				downOption: {
					auto: false //是否在初始化后,自动执行下拉回调callback; 默认true
				},
				upOption: {
					noMoreSize: 5, //如果列表已无数据,可设置列表的总数量要大于半页才显示无更多数据;避免列表数据过少(比如只有一条数据),显示无更多数据会不好看; 默认5
					textNoMore: '~ 没有更多了 ~',
				},
				hasNext: true,
				searchForm:{},
				CustomBar: '0'
			};
		},
		onShow() {
			this.hasNext = true
			// 重新加载数据
			if (this.mescroll) this.mescroll.resetUpScroll()
		},
		onLoad() {
			this.hasNext = true
			// 重新加载数据
			if (this.mescroll) this.mescroll.resetUpScroll()
		},
		methods: {
			// mescroll组件初始化的回调,可获取到mescroll对象
			mescrollInit(mescroll) {
				this.mescroll = mescroll;
			},
			/*下拉刷新的回调 */
			downCallback(mescroll) {
				this.hasNext = true
				// 重置分页参数页数为1
				mescroll.resetUpScroll()
			},
			/*上拉加载的回调: mescroll携带page的参数, 其中num:当前页 从1开始, size:每页数据条数,默认10 */
			async upCallback(mescroll) {
				let res = await this.$api.list(`users`, {
					page: mescroll.num,
					limit: mescroll.size
				});
				// 如果是第一页数据置空
				if (mescroll.num == 1) this.list = [];
				this.list = this.list.concat(res.data.list);
				if (res.data.list.length == 0) this.hasNext = false;
				mescroll.endSuccess(mescroll.size, this.hasNext);
			},
			// 详情
			onDetailTap(item) {
								this.$utils.jump(`./detail?id=${item.id}`)
							},
			// 修改
			onUpdateTap(id){
				this.$utils.jump(`./add-or-update?id=${id}`)
			},
			// 添加
			onAddTap(){
				this.$utils.jump(`./add-or-update`)
			},
			onDeleteTap(id){
				var _this = this;
				uni.showModal({
					title: '提示',
					content: '是否确认删除',
					success: async function(res) {
						if (res.confirm) {
							await _this.$api.del('users', JSON.stringify([id]));
							_this.hasNext = true
							// 重置分页参数页数为1
							_this.mescroll.resetUpScroll()
						}
					}
				});
			},
			// 搜索
			async search(){
				this.mescroll.num = 1
				let searchForm = {
					page: this.mescroll.num,
					limit: this.mescroll.size
				}
												if(this.searchForm.username){
					searchForm['username'] = '%' + this.searchForm.username + '%'
				}	
																												let res = await this.$api.list(`users`, searchForm);
				// 如果是第一页数据置空
				if (this.mescroll.num == 1) this.list = [];
				this.list = this.list.concat(res.data.list);
				if (res.data.list.length == 0) this.hasNext = false;
				this.mescroll.endSuccess(this.mescroll.size, this.hasNext);
			}
		}
	};
</script>

<style>
	/* product */
	page {
		background: #EEEEEE;
	}

	view {
		font-size: 28upx;
	}

	.uni-product-list {
		display: flex;
		width: 100%;
		flex-wrap: wrap;
		flex-direction: row;
		margin-top: 0upx;
	}

	.uni-product {
		padding: 10upx;
		margin: 10upx;
		display: flex;
		flex-direction: column;
		background: #FFFFFF;
	}

	.image-view {
		height: 330upx;
		width: 330upx;
		margin: 12upx 0;
	}

	.uni-product-image {
		height: 330upx;
		width: 330upx;
	}

	.uni-product-title {
		width: 300upx;
		word-break: break-all;
		display: -webkit-box;
		overflow: hidden;
		line-height: 1.5;
		text-overflow: ellipsis;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
	}

	.uni-product-price {
		margin-top: 10upx;
		font-size: 28upx;
		line-height: 1.5;
		position: relative;
	}

	.uni-product-price-original {
		color: #e80080;
	}

	.uni-product-price-favour {
		color: #888888;
		text-decoration: line-through;
		margin-left: 10upx;
	}

	.uni-product-tip {
		position: absolute;
		right: 10upx;
		background-color: #ff3333;
		color: #ffffff;
		padding: 0 10upx;
		border-radius: 5upx;
	}

	uni-image > div, uni-image > img {
		width: 100%;
		height: 140px;
		object-fit: cover;
	}

	.add-btn {
		position: fixed;
		left: 30upx;
		right: 30upx;
		// #ifndef MP
		bottom: 106upx;
		// #endif
		// #ifdef MP-WEIXIN
		bottom: 16upx;
		// #endif
		z-index: 95;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 690upx;
		height: 80upx;
		font-size: 32upx;
		color: #fff;
		background-color: red;
		border-radius: 10upx;
		box-shadow: 1px 2px 5px rgba(219, 63, 96, 0.4);
	}
	
	.list {
		padding: 20upx 20upx 20upx;
	}
	
	.listm {
		background: #fff;
		border-radius: 15upx;
		box-shadow: 0 0 2upx rgba(0, 0, 0, 0.1);
		margin-bottom: 20upx;
		padding: 30upx;
	}
	
	.listmpic {
		border-radius: 10upx;
		width: 166upx;
		margin-right: 20upx;
	}
	
	.listmr {
		// width: 460upx;
		display: inline-block;
		flex: 1;
		display: flex;
		justify-content: space-between;
		flex-direction: column;
	}
</style>
