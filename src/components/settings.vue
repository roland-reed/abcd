<template>
	<div>
		<div class="settings-container">
			<div class="setting-item">
				<div class="label">Auto play</div>
				<div
				class="switch-container"
				:class="autoPlayClass"
				@click="switchState('autoPlay')">
					<div class="switch-background">
						<div class="switch-background-active"></div>
					</div>
					<div class="switch-button"></div>
				</div>
				<p class="setting-item-detail">Audio will automatically be played when you expand the news if the switch is on.</p>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'settings',
		data() {
			return {
				autoPlay: false,
				autoPlayClass: null
			};
		},
		computed: {
			autoPlayClass() {
				return this.autoPlay ? {
					'switch-container-on': true
				} : null;
			}
		},
		methods: {
			switchState(key) {
				this[key] = !this[key];
				window.localStorage[key] = this[key];
				window.eventHub.$emit('updateautoplay');
			}
		},
		created() {
			if (window.localStorage.autoPlay === 'true') {
				this.autoPlay = true;
			} else if (window.localStorage.autoPlay === 'false') {
				this.autoPlay = false;
			}
		}
	};
</script>

<style lang="less">
	.settings-container {
		color: #666;
		margin-top: -40px;
	}
	.setting-item {
		position: relative;
		border: 1px  solid rgba(0,0,0,.075);
		padding: 0 16px;
		height: 40px;
		line-height: 40px;
		border-left: none;
		border-right: none;
	}
	.switch-container {
		position: absolute;
		right: 12px;
		top: 0;
		width: 50px;
		height: 40px;
		padding: 0 4px;
		overflow: hidden;
		-webkit-tap-highlight-color: transparent;
		cursor: pointer;
	}
	.switch-background {
		position: relative;
		margin: 5px 0;
		width: 50px;
		height: 30px;
		background-color: #fbfcfb;
		border: 1px solid rgba(0,0,0,.1);
		box-sizing: border-box;
		border-radius: 15px;
		box-shadow: 0 0 4px rgba(0,0,0,.2) inset;
		overflow: hidden;
		z-index: 5;
	}
	.switch-background-active {
		position: absolute;
		top: 0;
		left: -30px;
		width: 48px;
		height: 28px;
		border-radius: 15px;
		background-color: #a0c0bb;
		box-shadow: 0 0 4px rgba(0,0,0,.2) inset;
		transition: left .3s ease;
		z-index: 0;
	}
	.switch-button {
		position: absolute;
		left: 5px;
		top: 6px;
		border-radius: 50%;
		width: 28px;
		height: 28px;
		background-color: #fbfcfb;
		border: 1px solid rgba(0,0,0,.1);
		box-sizing: border-box;
		z-index: 5;
		transition: all .3s ease;
		box-shadow: 0 2px 4px rgba(0,0,0,.15);
	}
	.switch-container-on {
		.switch-button {
			left: 25px;
		}
		.switch-background-active {
			left: 0;
		}
	}
	.setting-item-detail {
		line-height: 1.5;
		font-size: 12px;
		color: #999;
		padding: 6px 0;
	}
</style>