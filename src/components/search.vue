<template>
	<div class="filter">
		<news-list
		:news-list="newsList"
		:if-show-message="ifShowMessage"
		:load-message="loadMessage"></news-list>

		<div
		id="search-load-more"
		v-show="ifShowLoadMore"
		class="load-more">{{loadMoreMessage}}</div>
	</div>
</template>

<script>
	import newsList from './news-list';
	import qwest from 'qwest';

	export default {
		name: 'search',
		components: {
			newsList
		},
		mounted: function() {
			window.eventHub.$on('searchinput', text => {
				this.q = text;
				this.changed = true;
			});

			window.eventHub.$on('search', this.search);

			let loadMoreNode = document.querySelector('#search-load-more');

			// 若加载更多按钮在可视范围之内则加载更多新闻
			window.eventHub.$on('scroll', () => {
				if (loadMoreNode.getBoundingClientRect().bottom < document.documentElement.clientHeight && loadMoreNode.getBoundingClientRect().top > 0) {
					this.loadMore();
				}
			});
		},
		data() {
			return {
				q: '',
				offset: 0,
				ifShowMessage: true,
				ifShowLoadMore: false,
				changed: false,
				isLoading: false,
				newsList: [],
				loadMoreMessage: 'Load More',
				loadMessage: 'Type keywords to search.'
			};
		},
		methods: {
			search() {
				if (this.q === '') {
					return;
				}

				if (!this.changed) {
					this.offset += 25;
				} else {
					this.offset = 0;
				}

				this.loadMessage = 'Searching...Please wait for a moment';
				this.ifShowMessage = true;
				this.newsList = [];
				this.changed = false;
				this.ifShowLoadMore = false;

				qwest.get('/abcd/search', {
						q: this.q,
						offset: this.offset
					}, {
						timeout: 10000
					})
					.then((xhr, res) => {
						if (res.code === 0) {
							if (res.news.length > 0) {
								this.newsList = res.news;
								this.ifShowMessage = false;
								this.ifShowLoadMore = true;
							} else {
								this.loadMessage = 'No news found.';
								this.ifShowLoadMore = false;
							}
						} else {
							this.loadMessage = 'Server error, please try again later';
							this.ifShowMessage = true;
						}
					})
					.catch(() => {
						this.loadMessage = 'No network.';
					});
			},
			loadMore() {
				if (this.isLoading) {
					return;
				}
				this.isLoading = true;
				this.loadMoreMessage = 'Loading...';
				this.offset = this.offset + 25;

				qwest.get('/abcd/search', {
						q: this.q,
						offset: this.offset
					})
					.then((xhr, res) => {
						if (res.code === 0){
							if (res.news.length > 0) {
								this.newsList = this.newsList.concat(res.news);
								this.loadMoreMessage = 'Load More';
							} else {
								this.loadMoreMessage = 'No more news.';
								this.isLoading = true;
							}
						} else {
							this.loadMessage = 'Server error, please try again later';
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

<style lang="less">
	.load-more {
		margin: 0 12px;
		line-height: 40px;
		height: 40px;
		background-color: rgba(160,192,187,1);
		text-align: center;
		color: #fff;
		margin-top: 12px;
	}
</style>