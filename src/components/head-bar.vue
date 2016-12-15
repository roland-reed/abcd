<template>
	<div class="head-container" :class="headContainerClass">
		<div class="head">
			<div class="menu-container" @click="toggleSidebar">
				<div class="menu icon-container">
					<i class="icon-menu"></i>
				</div>
			</div>

			<div class="title-container">
				<div class="title" :class="titleClass">
					{{title}}
				</div>
				<div class="audio-info">
					<span>
					{{audio.played | timeformat}} / {{audio.duration | timeformat}}
					</span>
				</div>
			</div>

			<div class="player-container">
				<div class="audio-control icon-container" @click="playOrPause">
					<i :class="playButtonClass"></i>
				</div>
			</div>
		</div>

		<div class="progress-bar" @click="setProgress">
			<div class="buffered" :style="bufferedBarStyle"></div>
			<div class="played" :style="progressBarStyle"></div>
			<div
			class="progress-dot"
			:style="progressDotStyle"
			@touchstart="touchSeek"
			@touchmove="touchSeek"
			@touchend="touchSeek"
			@touchcancel="touchSeek"></div>
		</div>

		<div
		class="source-list-container"
		:clientType="clientType"
		v-if="ifShowSubNav">
			<ul
			id="source-list"
			class="source-list"
			v-show="!ifShowSearch"
			@wheel="wheel">
				<li
				v-for="source in sourceList"
				:class="source.size"
				@click="changeSource($event, source.id)">{{source.name}}</li>
			</ul>
		</div>

		<div class="search-container" v-show="ifShowSubNav && ifShowSearch">
			<form class="search" action="#" @submit="submit($event)">
				<input
				type="search"
				placeholder="Search......(Only in title)"
				@change="emitInput($event)">
			</form>
		</div>
	</div>
</template>

<script>
	import Vue from 'vue';

	Vue.filter('timeformat', function(duration) {
		let mins = 0;
		let seconds = 0;

		mins = Math.floor(duration / 60);
		seconds = Math.floor(duration % 60);

		if (mins < 10) {
			mins = '0' + mins;
		}

		if (seconds < 10) {
			seconds = '0' + seconds;
		}

		return mins.toString() + ':' + seconds.toString();
	});

	export default {
		name: 'head-bar',
		data() {
			return {
				startX: 0,
				// 是否正在音频上调整进度
				seeking: false,
				// 是否正在播放音频
				audioPlaying: false,
				// 当前设备类型，默认为移动设备
				clientType: 'mobile',
				currentSourceId: 0,
				clientWidth: document.documentElement.clientWidth,
				playButtonClass: {
					'icon-play': true,
					'icon-pause': false,
					'icon-loading': false
				},
				sourceList: [{
					name: 'ABC',
					id: 0,
					size: {
						'source-li-current': true
					}
				}, {
					name: 'AP News',
					id: 1,
					size: {
						'middle-size': true
					}
				}, {
					name: 'BBC',
					id: 2
				}, {
					name: 'CNN',
					id: 3
				}, {
					name: 'NPR',
					id: 4
				}, {
					name: 'PBS',
					id: 5
				}, {
					name: 'VOA Normal',
					id: 6,
					size: {
						'large-size': true
					}
				}, {
					name: 'VOA Special',
					id: 7,
					size: {
						'large-size': true
					}
				}],
				audio: {
					instance: null,
					src: null,
					duration: 0,
					played: 0
				},
				progressDotStyle: {
					'left': '0'
				},
				progressBarStyle: {
					width: '0%'
				},
				bufferedBarStyle: {
					width: '0%'
				}
			};
		},
		computed: {
			playButtonClass: function() {
				return {
					'icon-play': !this.audioPlaying,
					'icon-pause': this.audioPlaying
				};
			},
			titleClass: function() {
				return this.audioPlaying ? {
					'title-small': true
				} : null;
			},
			headContainerClass: function() {
				return this.ifShowSubNav ? null : {
					'head-container-collapse': true
				};
			}
		},
		created: function() {
			this.clientWidth = document.documentElement.clientWidth;

			window.addEventListener('resize', () => {
				this.clientWidth = document.documentElement.clientWidth;
			});

			// 指示当前设备类型
			this.clientType = 'createTouch' in document ? 'mobile' : 'pc';

			// 监听loadAudio事件
			window.eventHub.$on('loadAudio', src => {
				this.loadAudio(src);
			});
		},
		mounted: function() {
			this.initAudio();
		},
		props: {
			ifShowSidebar: {
				type: Boolean,
				required: true,
				twoWay: true
			},
			toggleSidebar: {
				type: Function,
				required: true
			},
			title: {
				type: String,
				required: true
			},
			ifShowSearch: {
				type: Boolean,
				required: true
			},
			ifShowSubNav: {
				type: Boolean,
				required: true
			}
		},
		methods: {
			loadAudio: function(src) {
				this.audio.instance.src = src;
				this.playOrPause();
			},
			emitInput: function(e) {
				window.eventHub.$emit('searchinput', e.target.value);
			},
			submit: function(e) {
				e.preventDefault();
				window.eventHub.$emit('search');
			},
			changeSource: function(e, sourceId) {
				this.sourceList[window.state.sourceId].size['source-li-current'] = false;

				if (this.sourceList[sourceId].size) {
					this.sourceList[sourceId].size['source-li-current'] = true;
				} else {
					Vue.set(this.sourceList[sourceId], 'size', {
						'source-li-current': true
					});
				}

				window.state.sourceId = sourceId;

				window.eventHub.$emit('changesource');
			},
			initAudio: function() {
				let audio = new Audio();

				this.audio.instance = audio;

				audio.addEventListener('durationchange', () => {
					this.audio.instance = audio;
					this.audio.duration = audio.duration;
				});

				audio.addEventListener('progress', () => {
					if (audio.buffered.length <= 0) return;

					let buffered = audio.buffered.end(audio.buffered.length - 1);
					let percent = buffered / this.audio.duration;

					this.bufferedBarStyle['width'] = (percent * 100) + '%';
				});

				audio.addEventListener('play', () => {
					this.audioPlaying = true;
				});

				audio.addEventListener('pause', () => {
					this.audioPlaying = false;
				});
			},
			playOrPause: function() {
				let audio = this.audio.instance;

				if (!audio.src) {
					return;
				}

				if (audio.paused) {
					this.startPlayAudio();
				} else {
					audio.pause();
				}
			},
			startPlayAudio: function() {
				let audio = this.audio.instance;

				audio.play();

				// 设置一个每100ms更新一次进度条和进度点位置的定时器
				let playedId = setInterval(() => {
					if (this.seeking) {
						return;
					}

					let played = audio.currentTime;
					let position = 0;
					let percent = played / this.audio.duration;

					this.audio.played = played;
					position = percent * (this.clientWidth - 20);
					this.progressBarStyle['width'] = position + 10 + 'px';
					this.progressDotStyle['left'] = position + 'px';
				}, 100);

				// 在音频播放结束以后清除定时器
				audio.addEventListener('ended', () => {
					clearInterval(playedId);
				});
			},
			touchSeek: function(e) {
				let type = e.type;

				switch (type) {
				case 'touchstart':
					this.startX = e.touches[0].clientX;
					this.seeking = true;
					break;

				case 'touchmove':
					e.preventDefault();
					let audio = this.audio.instance;

					if (!audio) {
						return;
					}

					// 手指移动的距离
					let moved = e.touches[0].clientX - this.startX;
					// 手指移动的距离占屏幕的百分比
					let percent = moved / this.clientWidth;

					// 限制百分比的大小
					if (percent > 1) {
						percent = 1;
					}
					if (percent < -1) {
						percent = -1;
					}

					// 减去的20是进度点的宽度
					let movedPosition = percent * (this.clientWidth - 20);
					let movedDuration = percent * audio.duration;
					// 最终设置音频的时间
					let duration = 0;
					// 计算出当前进度点在屏幕上的位置
					let currentPosition = (audio.currentTime / audio.duration) * (this.clientWidth - 20);
					// 最终进度点和进度条的位置
					let position = 0;

					// 加上手指滑动距离之后的位置
					if (currentPosition + movedPosition < 0) {
						position = 0;
					} else if (currentPosition + movedPosition > this.clientWidth - 20) {
						position = this.clientWidth - 20;
					} else {
						position = currentPosition + movedPosition;
					}

					duration = audio.currentTime + movedDuration;
					this.progressBarStyle['width'] = position + 10 + 'px';
					this.progressDotStyle['left'] = position + 'px';
					this.movedPosition = duration;
					this.audio.played = duration;
					break;

				case 'touchend':
					this.audio.instance.currentTime = this.movedPosition;
					this.seeking = false;
					break;

				case 'touchcancel':
					this.seeking = false;
					break;
				}
			},
			setProgress: function(e) {
				let seekable = this.audio.instance.seekable;

				if (seekable.length < 1) {
					return;
				}

				let percent = e.clientX / this.clientWidth;
				let position = percent * (this.clientWidth - 20);

				if (percent * this.audio.duration > seekable.end(seekable.length - 1)) {
					return;
				}

				this.progressBarStyle['width'] = position + 10 + 'px';
				this.progressDotStyle['left'] = position + 'px';
				this.audio.played = percent * this.audio.duration;
				this.audio.instance.currentTime = percent * this.audio.duration;
			},
			wheel: function(e) {
				// 如果用户代理不支持触摸且窗口宽度不够显示所有课程类型
				// 则当鼠标在课程类型上滚动鼠标滚轮，就可以前后滑动类型列表，显示被隐藏的课程类型
				let tab = document.querySelector('#source-list');
				let position = window.getComputedStyle(tab, null).getPropertyValue('transform');

				if (document.documentElement.clientWidth < 775) {
					e.preventDefault();
					position = position.match(/\-?\d+/g);
					position = position ? +position[4] : 0;
					// 限制位移不超过-455px
					// 限制位移不超过0px
					position = (e.deltaY > 0) ? ((position - 150 <= -(775 - this.clientWidth)) ? -(775 - this.clientWidth) : position - 150) : ((position + 150 >= 0) ? 0 : position + 150);
					tab.style.transform = 'translateX(' + position + 'px)';
				}
			}
		}
	};
</script>

<style lang="less" scoped>
	.head-container {
		position: fixed;
		width: 100%;
		height: 102px;
		background-color: transparent;
		font-family: 'NewYorkIrvin';
		z-index: 10;
		box-shadow: 0 2px 8px rgba(0,0,0,.15);
	}
	.head-container-collapse {
		height: 60px;
	}
	.head {
		display: -webkit-flex;
		display: flex;
		flex-direction: row;
		height: 56px;
		background-color: #a0c0bb;
	}
	.menu-container {
		width: 56px;
		height: 56px;
	}
	.icon-container {
		width: 42px;
		height: 42px;
		margin-left: 7px;
		margin-top: 7px;
		border-radius: 50%;
		transition: all .3s ease;
		cursor: pointer;
		&:active, &:hover {
			background-color: rgba(0,0,0,0.1);
		}
		i {
			display: inline-block;
			width: 32px;
			height: 32px;
			margin-left: 5px;
			margin-top: 5px;
			font-size: 26px;
			line-height: 32px;
			text-align: center;
			text-align: center;
			color: #fff;
			vertical-align: middle;
		}
	}
	.title-container {
		width: 56px;
		height: 56px;
		flex-grow: 1;
		overflow: hidden;
	}
	.title {
		position: relative;
		color: #fff;
		font-size: 24px;
		height: 56px;
		line-height: 56px;
		transition: all .3s ease;
		background-color: #a0c0bb;
		z-index: 1;
	}
	.title-small {
		font-size: 18px;
		height: 40px;
		line-height: 40px;
	}
	.audio-info {
		font-size: 10px;
		height: 16px;
		line-height: 16px;
		overflow: hidden;
		word-break: break-all;
		color: #fff;
		span {
			&:first-child {
				padding-right: 6px;
			}
		}
	}
	.player-container {
		width: 56px;
		height: 56px;
	}
	.progress-bar {
		position: relative;
		width: 100%;
		height: 4px;
		cursor: pointer;
		background-color: #fff;
		transition: all .3s ease;
		-webkit-tap-highlight-color: transparent;
	}
	.source-list-container {
		background-color: #f4f4f4;
		width: 100%;
	}
	.source-list-container[clientType="mobile"] {
		overflow-x: scroll;
		-webkit-overflow-scrolling: touch;
	}
	.source-list {
		width: 775px;
		height: 42px;
		text-align: center;
		color: #a0c0bb;
		background-color: transparent;
		display: -webkit-flex;
		display: flex;
		transition: all .3s ease;
		font-size: 18px;
		.middle-size {
			width: 100px;
		}
		.large-size {
			width: 150px;
		}
		li {
			width: 75px;
			line-height: 42px;
			transition: all .3s ease;
			cursor: pointer;
			-webkit-tap-highlight-color: transparent;
			&:active, &:hover {
				color: #fff;
				background-color: #a0c0bb;
			}
		}
	}
	.source-li-current {
		background-color: #a0c0bb;
		color: #fff;
	}
	.buffered {
		position: absolute;
		left: 0;
		top: 0;
		width: 80%;
		height: 4px;
		background-color: rgba(99,214,201,.5);
	}
	.progress-dot {
		position: absolute;
		left: -7px;
		top: -8px;
		width: 20px;
		height: 20px;
		background-color: #fff;
		border-radius: 50%;
		border: 3px solid #a0c0bb;
		box-sizing: border-box;
		z-index: 2;
		transform: scale(1);
		box-shadow: 2px 2px 8px rgba(0,0,0,.15);
		transition: box-shadow .3s ease;
		&:hover {
			box-shadow: 2px 2px 8px rgba(0,0,0,.25), 2px 2px 4px rgba(0,0,0,.15);
		}
	}
	.played {
		position: absolute;
		left: 0;
		top: 0;
		width: 50%;
		height: 4px;
		background-color: #63d6c9;
	}
	.search-container {
		width: 100%;
		max-width: 660px;
		height: 42px;
		margin: 0 auto;
		padding: 8px 0;
		box-sizing: border-box;
		background-color: #fbfcfb;
	}
	.ios {
		.search-container {
			background-color: rgba(255,255,255,.5);
			-webkit-backdrop-filter: blur(5px);
		}
	}
	.search {
		height: 24px;
		line-height: 24px;
		input {
			width: 100%;
			padding: 0 16px;
			line-height: 24px;
			height: 24px;
			outline: 0;
			border: none;
			background-color: transparent;
			font-family: 'Hiragino Sans GB','Noto Sans CJK SC','Heiti SC','Microsoft Yahei','WenQuanYi Microhei', sans-serif;
			color: #333;
			font-size: 16px;
			box-sizing: border-box;
		}
	}
</style>