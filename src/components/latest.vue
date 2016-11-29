<template>
	<div>
		<news-list
		:news-list="newsList"
		:if-show-message="ifShowMessage"
		:load-message="loadMessage"></news-list>
		<div
		id="latest-load-more"
		v-show="ifShowLoadMore"
		class="load-more">{{loadMoreMessage}}</div>
	</div>
</template>

<script>
	import newsList from './news-list';
	import qwest from 'qwest';
	import config from '../config/config';

	export default {
		name: 'latest',
		components: {
			newsList
		},
		created() {
			window.eventHub.$on('changeview', view => {
				this.currentView = view;
			});

			window.eventHub.$on('changesource', sourceId => {
				if (this.currentView === 'latest') {
					this.isLoading = true;
					this.ifShowMessage = true;
					this.loadMessage = 'Loading...';
					this.loadNewsList(sourceId);
					this.sourceId = sourceId;
				}
			});

			this.loadNewsList(0);
		},
		mounted() {
			let loadMoreNode = document.querySelector('#latest-load-more');

			window.eventHub.$on('scroll', () => {
				if (loadMoreNode.getBoundingClientRect().bottom < document.documentElement.clientHeight && loadMoreNode.getBoundingClientRect().top > 0) {
					this.loadMore();
				}
			});
		},
		data() {
			return {
				currentView: 'latest',
				viewName: 'latest',
				ifShowMessage: true,
				ifShowLoadMore: false,
				loadMessage: config.msg.loading,
				loadMoreMessage: config.loadMore,
				isLoading: false,
				sourceId: 0,
				newsList: []
			};
		},
		methods: {
			loadNewsList(sourceId) {
				this.newsList = [];
				this.ifShowLoadMore = false;

				// 若上一个请求未结束则先结束上一个请求
				if (this.runningXHR && this.runningXHR.readyState !== 4) {
					this.runningXHR.abort();
				}

				this.runningXHR = qwest.get(`/abcd/latest/${sourceId}`, null, {
						timeout: config.timeout
					})
					.then((xhr, res) => {
						if (res.code === 0) {
							this.ifShowMessage = false;
							this.newsList = res.news;
							this.ifShowLoadMore = true;
						} else {
							this.loadMessage = config.msg.networkError;
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.loadMessage = config.msg.networkError;
						this.ifShowMessage = true;
						this.ifShowLoadMore = false;
					})
					.complete(() => {
						this.isLoading = false;
						window.eventHub.$emit('loaded');
					});
			},
			loadMore() {
				if (this.isLoading) {
					return;
				}
				this.isLoading = true;
				this.loadMoreMessage = config.msg.loading;

				qwest.get(`/abcd/latest/${this.sourceId}`, {
						offset: this.newsList.length
					}, {
						timeout: config.timeout
					})
					.then((xhr, res) => {
						if (res.code === 0) {
							if (res.news.length > 0) {
								this.newsList = this.newsList.concat(res.news);
								this.loadMoreMessage = config.msg.loadMore;
							} else {
								this.loadMoreMessage = config.msg.noMoreNews;
								this.isLoading = true;
							}
						} else {
							this.loadMessage = config.msg.networkError;
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.loadMoreMessage = config.msg.networkError;
					})
					.complete(() => {
						this.isLoading = false;
					});
			}
		}
	};
</script>