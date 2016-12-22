<template>
	<li @click="showMore" :style="newsStyle" :class="newsClass">
		<h3>
			<span class="bilingual-indicating">{{bilingualIndicating}}</span>
			{{title}}
		</h3>

		<div class="news-detail">
			<p>Source: <span>{{source}}</span></p>
			<p>Date: <span>{{date}}</span></p>
		</div>

		<div class="news-preview" :class="previewClass" v-show="ifShowPreview" v-html="previewContent"></div>

		<div class="news-preview news-content" v-show="!ifShowPreview" @mouseup="toggleChinese" :data-news-id="id">
			<div class="news-tip" v-show="ifShowTip">
				<p>Tap or click in the area of passage can toggle between Chinese and English if Chinese passage provided.</p>
				<p class="news-tip-close" @click="closeTip">Dont't show again.</p>
			</div>
			<div v-if="!ifShowChinese" v-html="englishContent" class="english-content"></div>
			<div v-if="ifShowChinese" v-html="chineseContent"></div>
		</div>

		<div class="news-methods" :class="methodsClass">
			<div>
				<div>
					<button v-show="ifShowPreview">EXPAND</button>
					<button v-show="!ifShowPreview" @click="showLess">COLLAPSE</button>
					<button :data-audio-src="audioSrc" @click="playAudio">PLAY AUDIO</button>
				</div>
			</div>
		</div>
	</li>
</template>

<script>
	import Vue from 'vue';

	export default {
		name: 'news',
		data() {
			return {
				ifShowPreview: true,
				ifShowChinese: false,
				methodsFloating: false,
				newsStyle: {
					'cursor': 'pointer'
				},
				newsClass: {
					'news-collapse': true
				},
				methodsClass: {
					'methods-floating': false
				}
			};
		},
		props: {
			id: {
				type: Number,
				required: true
			},
			title: {
				type: String,
				required: true
			},
			audioSrc: {
				type: String,
				required: false
			},
			source: {
				type: String,
				required: true
			},
			date: {
				type: String,
				required: true
			},
			chineseContent: {
				type: String,
				required: false
			},
			englishContent: {
				type: String,
				required: true
			},
			ifShowTip: {
				type: Boolean,
				required: true,
				twoWay: true
			}
		},
		mounted: function() {
			window.eventHub.$on('scroll', () => {
				let content = document.querySelector(`.news-content[data-news-id="${this.id}"]`);
				let clientHeight = document.documentElement.clientHeight;

				// 如果内容在可视范围之内则将操作栏显示在屏幕下方
				if (content && content.getBoundingClientRect().top < clientHeight && content.getBoundingClientRect().bottom > clientHeight) {
					this.methodsFloating = true;
				} else {
					this.methodsFloating = false;
				}
			});

			window.eventHub.$on('updateautoplay', () => {
				this.setAutoPlay();
			});

			// 设置是否自动播放
			this.setAutoPlay();
		},
		computed: {
			newsStyle() {
				return this.ifShowPreview ? {
					'cursor': 'pointer'
				} : {
					'cursor': 'auto'
				};
			},
			newsClass() {
				return this.ifShowPreview ? {
					'news-collapse': true
				} : {
					'news-collapse': false
				};
			},
			methodsClass() {
				return this.methodsFloating ? {
					'methods-floating': true
				} : {
					'methods-floating': false
				};
			},
			previewClass() {
				return !this.chineseContent ? {
					'preview-content': true
				} : null;
			},
			previewContent() {
				if (this.chineseContent) {
					let node = document.createElement('p');
					node.innerHTML = this.chineseContent;
					return node.innerText.slice(0, 100) + '······';
				} else {
					let node = document.createElement('p');
					node.innerHTML = this.englishContent;
					return node.innerText.length > 240 ? (node.innerText.slice(0, 240) + '......') : node.innerText;
				}
			},
			bilingualIndicating() {
				return this.chineseContent ? '【双语】' : '【仅英文】';
			}
		},
		methods: {
			showMore() {
				this.ifShowPreview = false;
				this.methodsFloating = true;

				// 触发视图更新，若不需浮动操作栏则清除浮动操作栏
				Vue.nextTick(() => {
					window.eventHub.$emit('scroll');
				});
				
				if (this.autoPlay) {
					window.eventHub.$emit('loadAudio', this.audioSrc);
				}
			},
			showLess(e) {
				e.stopPropagation();
				this.ifShowPreview = true;
				this.methodsFloating = false;
			},
			toggleChinese(e) {
				e.stopPropagation();
				if (this.chineseContent && !window.getSelection().toString()) {
					this.ifShowChinese = !this.ifShowChinese;
				}
			},
			playAudio(e) {
				e.stopPropagation();
				window.eventHub.$emit('loadAudio', this.audioSrc);
			},
			closeTip() {
				window.eventHub.$emit('closeTip');
			},
			setAutoPlay() {
				if (window.localStorage.autoPlay === 'true') {
					this.autoPlay = true;
				} else if (window.localStorage.autoPlay === 'false') {
					this.autoPlay = false;
				}
			}
		}
	};
</script>

<style lang="less">
	.bilingual-indicating {
		color: #a0c0bb;
		font-size: 15px;
		display: inline-block;
	}
	.news-detail {
		font-size: 12px;
		color: #999;
		overflow: hidden;
		margin-bottom: 1em;
		p {
			float: left;
			&:last-child {
				margin-left: 12px;
			}
		}
	}
	.news-preview {
		font-size: 15px;
		color: #000;
		line-height: 1.6em;
		text-align: justify;
		p {
			margin-bottom: 1em;
			&>img {
				margin-bottom: 1em;
			}
		}
		img {
			width: auto;
			height: auto;
			max-width: 100%;
			display: block;
			margin: auto;
		}
	}
	.news-methods {
		float: right;
		button {
			-webkit-appearance: none;
			background-color: transparent;
			border: none;
			font-size: 16px;
			color: #a0c0bb;
			font-family: inherit;
			height: 40px;
			line-height: 40px;
			font-weight: bold;
			cursor: pointer;
			transition: all .3s ease;
			border-radius: 2px;
			padding: 0 12px;
			outline: none;
			font-family: NewYorkIrvin;
			&:hover {
				background-color: rgba(0,0,0,.075);
			}
		}
	}
	.news-tip {
		text-align: center;
		margin: 0 1em;
		color: #999;
		font-size: 12px;
		p {
			margin-bottom: 0;
		}
	}
	.news-tip-close {
		text-decoration: underline;
		cursor: pointer;
	}
	.news-content {
		font-size: 15px;
		line-height: 1.6em;
	}
	.english-content, .preview-content {
		font-family: 'MyriadSetPro', Helvetica, Arial;
	}
	.news-collapse {
		&:active {
			background-color: rgba(0,0,0,.05);
		}
	}
	.methods-floating {
		position: fixed;
		bottom: 0;
		right: 0;
		width: 100%;
		padding-right: 12px;
		background-color: #fbfcfb;
		box-shadow: 0 -2px 8px rgba(0,0,0,.15);
		&>div {
			max-width: 684px;
			margin: 0 auto;
			&>div {
				float: right;
			}
		}
	}
	.ios {
		.methods-floating {
			background-color: rgba(251,252,251,.5);
			-webkit-backdrop-filter: blur(10px);
		}
		.source-list-container {
			background-color: rgba(244,244,244,.5);
			-webkit-backdrop-filter: blur(10px);
		}
	}
</style>