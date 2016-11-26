<template>
	<div class="sidebar" :class="sidebarClass">
		<div class="sidebar-container">
			<div class="sidebar-title">{{title}}</div>
			<div class="sidebar-nav">
				<ul class="nav-list">
					<li @click="changeView('latest')">
						<span>
							<i class="icon-latest"></i>
						</span>
						<span>LATEST</span>
					</li>
					<li @click="changeView('archive')">
						<span>
							<i class="icon-archive"></i>
						</span>
						<span>ARCHIVE</span>
					</li>
					<li @click="changeView('search')">
						<span>
							<i class="icon-search"></i>
						</span>
						<span>SEARCH</span>
						</li>
					<li @click="changeView('settings')">
						<span>
							<i class="icon-settings"></i>
						</span>
						<span>SETTINGS</span>
					</li>
					<li @click="changeView('help')">
						<span>
							<i class="icon-help"></i>
						</span>
						<span>HELP</span>
					</li>
					<li @click="changeView('about')">
						<span>
							<i class="icon-about"></i>
						</span>
						<span>ABOUT</span>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'sidebar',
		data() {
			return {
				title: 'ABCD',
				sidebarClass: {
					'sidebar-show': false
				}
			};
		},
		props: {
			ifShowSidebar: {
				type: Boolean,
				required: true,
				twoWay: true
			}
		},
		computed: {
			sidebarClass: function() {
				return {
					'sidebar-show': this.ifShowSidebar
				};
			}
		},
		methods: {
			changeView(view) {
				window.eventHub.$emit('changeview', view, document.body.scrollTop);
			}
		}
	};
</script>

<style lang="less">
	.sidebar {
		position: fixed;
		width: 200px;
		height: 100%;
		background-color: #fbfcfb;
		font-family: 'NewYorkIrvin';
		z-index: 20;
		box-sizing: border-box;
		color: #636363;
		box-shadow: 0 4px 8px rgba(0,0,0,0.1), 0 2px 4px rgba(0,0,0,0.1);
		transform: translate(-220px,0);
		transition: all .3s ease;
		overflow: auto;
	}
	.ios {
		.sidebar {
			background-color: rgba(251,252,251,.9);
			-webkit-backdrop-filter: blur(10px);
		}
	}
	.sidebar-show {
		transform: translate(0,0);
	}
	.sidebar-title {
		height: 56px;
		line-height: 56px;
		font-size: 24px;
		text-indent: 24px;
		cursor: pointer;
	}
	.nav-list {
		li {
			height: 40px;
			line-height: 40px;
			font-size: 16px;
			text-indent: 24px;
			transition: all .3s ease;
			cursor: pointer;
			span {
				i {
					font-size: 20px;
					vertical-align: middle;
				}
				&:first-child {
					padding-right: 12px;
				}
			}
			&:hover {
				background-color: rgba(0,0,0,0.075);
			}
		}
	}
</style>