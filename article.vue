<template>
	<div v-if="pageInfo">
		<!-- 文章 -->
		<div class="charge-article">
			<div class="charge-article-title">{{pageInfo.detail.title}}</div>
			<div class="charge-article-list">
				<div :class="{'charge-article-item':true, 'active': tabStatus == 1}" @click="tabStatus=1">概念</div>
				<div :class="{'charge-article-item':true, 'active': tabStatus == 2}" @click="tabStatus=2">量刑标准</div>
				<div :class="{'charge-article-item':true, 'active': tabStatus == 3}" @click="tabStatus=3">立案标准</div>
			</div>
			<!-- <div class="charge-title">概念</div> -->
			<div class="charge-article-article article-article" v-html="chargeArticle"></div>
			<!-- 统计信息  -->
			<div class="charge-article-msg">
				<div class="charge-msg-item">
					<i class="iconfont charge-item-icon" style="font-size: 20rpx;">&#xe627;</i>
					<div class="charge-item-span">{{pageInfo.detail.views}}</div>
				</div>
				<div :class="{'charge-msg-item': true, 'active': pageInfo.zan==1}" @click="zanClick">
					<i class="iconfont charge-item-icon">&#xe635;</i>
					<div :class="{'charge-item-span': true}">{{pageInfo.detail.zanNum}}</div>
				</div>
			</div>
		</div>
		<!-- banner -->
		<QuickOrders :answerNum="pageInfo.todayask"></QuickOrders>
		<!-- 相关法规 -->
		<LawsList :lawsTitle="'热门法规'" :lawsList="pageInfo.regulations"></LawsList>
		<!-- 律师推荐 -->
		<Lawyer :lawyerTitle="'推荐律师'" :titleStyle="titleStyle" :homeInfo="pageInfo.lvList"></Lawyer>
		<!-- 底部 -->
		<ChargeFooter></ChargeFooter>
		<!-- 点击置顶 -->
		<GoTop :bottomMargin="'150rpx'"></GoTop>
	</div>
</template>

<script>
	import {
		mapState,
		mapMutations
	} from 'vuex';
	import GoTop from '@/common/GoTop.vue';
	import Lawyer from '@/components/find/Lawyer.vue';
	import QuickOrders from '@/common/QuickOrders.vue';
	import LawsList from '@/components/find/LawsList.vue';
	import { getZmArticle, getZmZan } from '@/api/find.js'
	import ChargeFooter from '@/components/find/ChargeFooter.vue';
	export default {
		data() {
			return {
				pageInfo: '',
				tabStatus: 1,
				titleStyle: {
					'text-align': 'center',
					'width': '100%'
				},
			}
		},
		components: {
			GoTop,
			Lawyer,
			LawsList,
			ChargeFooter,
			QuickOrders,
		},
		computed: {
			...mapState(['api', 'source', 'openid', 'token', 'isShowOrderWin']),
			chargeArticle() {
				if(!this.pageInfo){
					return;
				}
				if (this.tabStatus == 1) {
					return this.pageInfo.detail.gn
				} else if (this.tabStatus == 2) {
					return this.pageInfo.detail.lxbz
				} else {
					return this.pageInfo.detail.labz
				}
			}
		},
		onLoad(option) {
			this.id = option.id;
			this.getPageInfo();
		},
		methods: {
			getPageInfo(){
				getZmArticle({
					method: 'POST',
					data: {
						bId: this.id,
						openid: this.openid,
						token: this.token,
					}
				}).then(res=>{
					this.pageInfo = res.data;
					// #ifdef MP-BAIDU
					swan.setPageInfo({
						...res.data.bdSeo,
						success: res => {
							console.log('set-bdSeo-res:', res);
						},
						fail: err=>{
							console.log('set-bdSeo-err:',err);
						}
					})
					// #endif
					uni.setNavigationBarTitle({
						title: res.data.detail.title
					});
				})
			},
			// 点赞
			zanClick() {
				getZmZan({
					method: 'POST',
					data: {
						bId: this.id,
						openid: this.openid,
						token: this.token,
					}
				}).then(res=>{
					if (res.code == 1) {
						uni.showToast({
							title: res.msg,
							icon: 'none'
						});
						this.pageInfo.zan = this.pageInfo.zan == 1 ? 0 : 1;
						this.pageInfo.detail.zanNum = this.pageInfo.zan == 1 ? ++this.pageInfo.detail.zanNum : --this.pageInfo.detail.zanNum
					} else {
						uni.showToast({
							title: res.msg,
							icon: 'none'
						});
					}
				})
			}
		}
	}
</script>

<style scoped="scoped">
	/* 面包屑 */
	.charge-mbx {
		display: flex;
		padding: 30rpx;
		font-size: 26rpx;
		font-weight: 400;
		color: rgba(64, 83, 118, 1);
	}

	.charge-mbx span {
		margin-right: 100rpx;
	}

	.charge-mbx .active {
		color: #2159FB;
	}

	/* 文章 */
	.charge-article {
		background-color: #FFF;
		padding: 30rpx;
		margin-bottom: 20rpx;
	}

	.charge-article-title {
		font-size: 32rpx;
		font-weight: bold;
		color: rgba(21, 41, 77, 1);
		line-height: 40rpx;
		margin-bottom: 33rpx;
	}

	.charge-article-list {
		display: flex;
		margin-bottom: 24rpx;
	}

	.charge-article-item {
		width: 157rpx;
		height: 58rpx;
		border: 1px solid rgba(136, 147, 169, 1);
		border-radius: 29rpx;
		font-size: 28rpx;
		font-weight: 400;
		color: rgba(137, 147, 169, 1);
		line-height: 40rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-right: 10rpx;
		margin-bottom: 16rpx;
	}

	.charge-article-item.active {
		border: 1px solid rgba(35, 89, 253, 1);
		color: rgba(35, 89, 253, 1);
	}

	.charge-title {
		font-size: 30rpx;
		font-weight: bold;
		color: rgba(21, 41, 77, 1);
		line-height: 40rpx;
		margin-bottom: 30rpx;
	}

	.charge-article-article {
		margin-bottom: 30rpx;
	}

	.charge-article-msg {
		display: flex;
	}

	.charge-msg-item {
		display: flex;
		align-items: center;
	}

	.charge-item-span {
		font-size: 24rpx;
		font-weight: 400;
		color: rgba(137, 155, 191, 1);
		margin-right: 60rpx;
	}

	.charge-item-icon {
		color: #8B97AB;
		margin-right: 10rpx;
	}

	.charge-msg-item.active .charge-item-icon,
	.charge-msg-item.active .charge-item-span {
		color: #225AFC !important;
	}
</style>
