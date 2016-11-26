<template>
	<div class="archive">
		<p class="load-message" v-show="ifShowMessage">{{this.loadMessage}}</p>
		<archive-list-item
		v-for="archive in archiveList"
		:archive="archive"
		:source-id="sourceId"
		:if-show-message="ifShowMessage"></archive-list-item>
	</div>
</template>

<script>
	import archiveListItem from './archive-list-item';
	import qwest from 'qwest';

	export default {
		name: 'archive',
		components: {
			archiveListItem
		},
		created() {
			window.eventHub.$on('changeview', view => {
				this.currentView = view;
			});

			// 当更换来源时重新载入新闻
			window.eventHub.$on('changesource', sourceId => {
				if (this.currentView === 'archive') {
					this.ifShowMessage = true;
					this.loadMessage = 'Loading archive infomation...';
					this.sourceId = sourceId;
					this.loadEarliestMonth();
				}
			});

			this.loadEarliestMonth();
		},
		data() {
			return {
				sourceId: 0,
				currentView: 'archive',
				ifShowMessage: true,
				loadMessage: 'Loading archive infomation...',
				archiveList: []
			};
		},
		methods: {
			loadEarliestMonth() {
				this.archiveList = [];

				qwest.get(`/abcd/archive/earliestMonth/${this.sourceId}`, null, {
						timeout: 3000
					})
					.then((xhr, res) => {
						if (!res.date) {
							this.loadMessage = 'Sorry, no archive found.';
							this.ifShowMessage = true;
							return;
						}

						let d = new Date();
						let currentYear = d.getFullYear();
						let currentMonth = d.getMonth();
						let startYear = res.date.year;
						let startMonth = res.date.month;
						let monthArray = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];

						for (let i = startYear; i <= currentYear; i++) {
							if (i === startYear) {
								for (let j = startMonth; j < 13; j++) {
									this.archiveList.unshift({
										year: i,
										month: j,
										monthLabel: `${monthArray[j - 1]}, ${i}`
									});
								}
							} else if (i === currentYear) {
								for (let j = 1; j < currentMonth + 2; j++) {
									this.archiveList.unshift({
										year: i,
										month: j,
										monthLabel: `${monthArray[j - 1]}, ${i}`
									});
								}
							} else {
								for (let j = 0; j < 12; j++) {
									this.archiveList.unshift({
										year: i,
										month: j,
										monthLabel: `${monthArray[j]}, ${i}`
									});
								}
							}
						}

						this.ifShowMessage = false;
					})
					.catch(() => {
						this.ifShowMessage = true;
						this.loadMessage = 'No network.';
					});
			}
		}
	};
</script>

<style lang="less">
	.archive {
		box-sizing: border-box;
		-webkit-tap-highlight-color: transparent;
		&>div {
			margin-top: 12px;
			&:first-child {
				margin-top: 0;
			}
		}
		h2 {
			color: #fff;
			margin: 0 12px;
			padding: 0 12px;
			background-color: #a0c0bb;
			line-height: 40px;
			cursor: pointer;
			margin-bottom: 12px;
			transition: all .3s ease;
			&:hover, &:active {
				background-color: darken(#a0c0bb, 10%);
			}
		}
	}
	.load-message {
		text-align: center;
		color: #999;
		padding: 0 12px;
	}
</style>