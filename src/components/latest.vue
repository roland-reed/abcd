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
				loadMessage: 'Loading...',
				loadMoreMessage: 'Load More',
				isLoading: false,
				sourceId: 0,
				newsList: []
			};
		},
		methods: {
			loadNewsList(sourceId) {
				this.newsList = [];
				this.ifShowLoadMore = false;

				qwest.get(`/abcd/latest/${sourceId}`, null, {
						timeout: 10000
					})
					.then((xhr, res) => {
						if (res.code === 0) {
							this.ifShowMessage = false;
							this.newsList = res.news;
							this.ifShowLoadMore = true;
							window.eventHub.$emit('loaded');
						} else {
							this.loadMessage = 'Server error, please try agian later.';
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.ifShowMessage = true;
						this.loadMessage = 'No network.';
						this.ifShowLoadMore = false;
					})
					.complete(() => {
						this.isLoading = false;
					});
			},
			loadMore() {
				if (this.isLoading) {
					return;
				}
				this.isLoading = true;
				this.loadMoreMessage = 'Loading...';

				qwest.get(`/abcd/latest/${this.sourceId}`, {
						offset: this.newsList.length
					})
					.then((xhr, res) => {
						if (res.code === 0) {
							if (res.news.length > 0) {
								this.newsList = this.newsList.concat(res.news);
								this.loadMoreMessage = 'Load More';
							} else {
								this.loadMoreMessage = 'No more news.';
								this.isLoading = true;
							}
						} else {
							this.loadMessage = 'Server error, please try agian later.';
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.loadMoreMessage = 'No network.';
					})
					.complete(() => {
						this.isLoading = false;
					});
			}
		}
	};
</script>