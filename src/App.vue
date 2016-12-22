<template>
	<div id="app" :class="appClass">
		<div class="start-page" v-show="!loaded">
			<div class="start-page-content">
				<h1>ABCD</h1>
				<div class="intro">Simple but powerful bilingual news collection.</div>
			</div>
			<div class="loading">
				loading...
			</div>
		</div>

		<transition name="fade">
			<div class="mask"
			@click="toggleSidebar"
			@touchmove="preventScroll"
			v-show="ifShowSidebar"></div>
		</transition>

		<head-bar
		:if-show-sidebar="this.ifShowSidebar"
		:if-show-sub-nav="ifShowSubNav"
		:toggle-sidebar="this.toggleSidebar"
		:title="currentView"
		:if-show-search="ifShowSearch"></head-bar>

		<sidebar :if-show-sidebar="this.ifShowSidebar"></sidebar>

		<div class="content-container">
			<keep-alive>
				<component :is="currentView"></component>
			</keep-alive>
		</div>
	</div>
</template>

<script>
	import Vue from 'vue';
	import HeadBar from './components/head-bar';
	import Sidebar from './components/sidebar';
	import latest from './components/latest';
	import archive from './components/archive';
	import search from './components/search';
	import settings from './components/settings';
	import help from './components/help';
	import about from './components/about';

	export default {
		name: 'app',
		components: {
			HeadBar,
			Sidebar,
			latest,
			archive,
			search,
			settings,
			help,
			about
		},
		created() {
			document.addEventListener('touchstart', function() {}, true);

			window.eventHub = new Vue();

			window.state = {
				sourceId: 0
			};

			window.addEventListener('scroll', (e) => {
				window.eventHub.$emit('scroll', e);
			});

			window.eventHub.$on('changeview', (view, scrollTop) => {
				this.scrollTop[this.currentView] = scrollTop;
				this.currentView = view;
				Vue.nextTick(() => {
					document.body.scrollTop = this.scrollTop[view];
				});
				this.toggleSidebar();
			});

			if (navigator.platform.match(/iphone|ipad|ipod|mac/i)) {
				this.safari = true;
			}
		},
		data() {
			return {
				ifShowSidebar: false,
				loaded: false,
				safari: false,
				currentView: 'latest',
				currentSourceId: 0,
				scrollTop: {
					latest: 0,
					archive: 0,
					search: 0,
					settings: 0,
					help: 0
				}
			};
		},
		computed: {
			appClass() {
				return this.safari ? {
					'ios': true
				} : null;
			},
			ifShowSearch() {
				return this.currentView === 'search';
			},
			ifShowSubNav() {
				return this.currentView !== 'settings' && this.currentView !== 'about' && this.currentView !== 'help';
			}
		},
		mounted() {
			window.eventHub.$on('loaded', () => {
				setTimeout(() => {
					this.loaded = true;
				}, 1000);
			});
		},
		methods: {
			toggleSidebar() {
				this.ifShowSidebar = !this.ifShowSidebar;
				
				let body = document.querySelector('body');

				if (this.ifShowSidebar) {
					body.style.overflow = 'hidden';
				} else {
					body.style.overflow = 'auto';
				}
			},
			preventScroll(e) {
				e.preventDefault();
				return;
			}
		}
	};
</script>

<style lang="less">
	@import 'style/reset.less';
	@import '../static/icons/style.less';

	@base-font: 'Hiragino Sans GB','Noto Sans CJK SC','Heiti SC','Microsoft Yahei','WenQuanYi Microhei', sans-serif;

	body {
		width: 100%;
		font-family: @base-font;
		background-color: #fbfcfb;
		padding-bottom: 40px;
	}
	.start-page {
		position: fixed;
		top: 0;
		left: 0;
		bottom: 0;
		right: 0;
		background-color: #fbfcfb;
		color: #a0c0bb;
		font-family: NewYorkIrvin;
		font-size: 16px;
		z-index: 100;
		.start-page-content {
			position: fixed;
			top: 50%;
			text-align: center;
			transform: translate(0, -50%);
			width: 100%;
			padding: 0 24px;
			box-sizing: border-box;
		}
		h1 {
			margin-bottom: 24px;
			font-size: 56px;
		}
		.loading {
			position: fixed;
			bottom: 48px;
			box-sizing: border-box;
			width: 100%;
			text-align: center;
		}
	}
	.mask {
		position: fixed;
		top: 0;
		left: 0;
		bottom: 0;
		right: 0;
		background-color: rgba(0,0,0,0.75);
		z-index: 15;
		opacity: 1;
		transition: all .3s ease;
		-webkit-tap-highlight-color: transparent;
	}
	.ios {
		.mask {
			-webkit-backdrop-filter: blur(5px);
			background-color: rgba(0,0,0,.5);
		}
	}
	.fade-enter-active, .fade-leave-active {
		transitionL: opacity .3s ease;
	}
	.fade-enter, .fade-leave-active {
		opacity: 0;
	}
	.content-container {
		width: 100%;
		max-width: 684px;
		margin: 0 auto;
		padding-top: 120px;
	}
</style>