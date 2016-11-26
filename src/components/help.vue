<template>
	<article class="passage">
		<section>
			<h3>What is ABCD?</h3>
			<p>
				Short version, ABCD is a bilingual news collection.
			</p>
			<p>
				Long version, ABCD is a simple but powerful bilingual news collection which sourced from ABC, AP News, BBC, CNN, NPR, VOA and etc.
			</p>
			<p>
				Although it's a bilingual news collection, many of them are only in english.
			</p>
		</section>
		<section>
			<h3>how to use?</h3>
			<p>
				Every news has a relevant audio, you can play it by simply tapping the <b><i>PLAY AUDIO</i></b> button.
			</p>
			<p>
				If a news is bilingual, tap or click on the passage will toggle between English and Chinese.
			</p>
			<p>
				The source list on the top navigation bar can change source in <b>LATEST</b> and <b>ARCHIVE</b> section, if you are using a PC or Mac and the source list cannot show all source options, you can move your cursor to the list and scroll your wheel to make the rest options available, if you are using a mobile device, you can simply use your finger sliding on the source list.
			</p>
			<p>
				The <b>LATEST</b> section offers you the latest news from every source, you can read the news and listen to the audio relevant.If you want more news, you can go to the <b>ARCHIVE</b> section.
			</p>
			<p>
				The <b>ARCHIVE</b> section saved lots of news from every source, the news are displayed in reverse chronological order by month. Tap or click on the month label can view all news in that month.
			</p>
		</section>
	</article>
</template>

<script>
	export default {
		name: 'help'
	};
</script>

<style lang="less">
	.passage {
		margin-top: -40px;
		font-family: 'MyriadSetPro', Helvetica, Arial;
		padding: 0 12px;
		section {
			h3 {
				padding: 12px 0;
				font-size: 24px;
				font-family: 'NewYorkIrvin';
			}
			p {
				line-height: 1.5em;
				margin-bottom: 1.5em;
				text-align: justify;
				a {
					color: #a0c0bb;
					text-decoration: none;
					&:link {
						color: #a0c0bb;
					}
					&:visited {
						color: #a0c0bb;
					}
					&:hover {
						color: #a0c0bb;
						border-bottom: 1px solid #a0c0bb;
					}
				}
			}
		}
	}
</style>