<template>
	<div class="archive-instance">
		<h2
		@click="toggleList"
		:class="monthLabelClass">{{archive.monthLabel}}</h2>

		<ul :id="contentId" v-show="showList" :class="contentClass">
			<news-list
			:news-list="newsList"
			:load-message="loadMessage"
			:if-show-message="ifShowMessage"></news-list>
		</ul>
	</div>
</template>

<script>
	import Vue from 'vue';
	import newsList from './news-list';
	import qwest from 'qwest';
	import config from '../config/config';

	export default {
		name: 'archive-list-item',
		components: {
			newsList
		},
		data() {
			return {
				showList: false,
				newsList: [],
				loaded: false,
				ifShowMessage: true,
				loadMessage: config.msg.newsLoading,
				monthLabelFloat: false
			};
		},
		computed: {
			contentId() {
				return `archive-title-${this.archive.year}-${this.archive.month}`;
			},
			monthLabelClass() {
				return this.monthLabelFloat ? {
					'month-label-float': true
				} : null;
			},
			contentClass() {
				return this.monthLabelFloat ? {
					'content-float-fix': true
				} : null;
			}
		},
		mounted() {
			this.contentEl = document.getElementById(this.contentId);

			// 监听页面的滚动事件
			// 若内容区域在可视范围之内则将折叠和播放操作浮动在窗口最下面
			window.eventHub.$on('scroll', () => {
				if (this.showList) {
					let position = this.contentEl.getBoundingClientRect();
					if (position.top < 102 && position.bottom > 142) {
						this.monthLabelFloat = true;
					} else {
						this.monthLabelFloat = false;
					}
				} else {
					this.monthLabelFloat = false;
				}
			});
		},
		props: {
			archive: {
				type: Object,
				required: true
			}
		},
		methods: {
			toggleList() {
				this.showList = !this.showList;

				// 如果内容在可视范围之内则不浮动显示月份标签
				Vue.nextTick(() => {
					window.eventHub.$emit('scroll');
				});

				if (!this.loaded) {
					let month = this.archive.month < 10 ? '0' + this.archive.month : this.archive.month;
					this.loadNewsList(window.state.sourceId, this.archive.year.toString() + '-' + month);
				}
			},
			loadNewsList(sourceId, month) {
				qwest.get(`/abcd/archive/${sourceId}/${month}`, null, {timeout: config.timeout})
					.then((xhr, res) => {
						this.loaded = true;
						this.ifShowMessage = false;

						if (res.code === 0 && res.news.length > 0) {
							this.newsList = res.news;
						} else {
							this.loadMessage = config.msg.archiveNoNews;
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.ifShowMessage = true;
						this.loadMessage = config.msg.networkError;
					});
			},
		}
	};
</script>

<style lang="less">
	.archive .month-label-float {
		position: fixed;
		top: 102px;
		width: 100%;
		max-width: 660px;
		margin: 0;
		box-shadow: 0 2px 8px rgba(0,0,0,.1);
		z-index: 13;
	}
	.content-float-fix {
		padding-top: 52px;
	}
</style>