<template>
	<div class="content">
		<p class="load-message" v-show="ifShowMessage">{{loadMessage}}</p>
		
		<ul class="news-list">
			<news v-for="news in newsList"
			:id="news.id"
			:title="news.title"
			:source="news.source"
			:date="news.date"
			:audio-src="news.mp3Url"
			:english-content="news.englishContent"
			:chinese-content="news.chineseContent"
			:if-show-tip="ifShowTip"></news>
		</ul>
	</div>
</template>

<script>
	import News from './news';

	export default {
		name: 'news-list',
		components: {
			News
		},
		created() {
			if (!window.localStorage.ifShowTip) {
				window.localStorage.ifShowTip = 1;
			} else {
				if (+window.localStorage.ifShowTip === 0) {
					this.ifShowTip = true;
				} else {
					this.ifShowTip = false;
				}
			}

			window.eventHub.$on('closeTip', () => {
				this.ifShowTip = false;
			});
		},
		data() {
			return {
				ifShowTip: false
			};
		},
		props: {
			newsList: {
				type: Array,
				required: false
			},
			ifShowMessage: {
				type: Boolean,
				required: true
			},
			loadMessage: {
				type: String,
				required: false
			}
		}
	};
</script>

<style lang="less">
	.news-list {
		li {
			cursor: pointer;
			overflow: hidden;
			margin-top: 12px;
			-webkit-tap-highlight-color: transparent;
			padding: 0 12px;
			&:first-child {
				margin-top: 0;
			}
			h3 {
				font-size: 24px;
				color: #000;
				padding: 8px 0;
				overflow: hidden;
			}
		}
	}
	.load-message {
		text-align: center;
		color: #999;
	}
</style>