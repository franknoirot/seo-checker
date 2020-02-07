<script>
	import { onMount } from 'svelte'
	let postText = ''
	let keywords = []
	let keywordsMatchCount = []

	const wordLength = 6
	$: wordCount = postText.split(' ').reduce((acc, word) => acc + ((word.length < wordLength) ? 1 : Math.floor(word.length / wordLength)), 0)
	$: postHTML = parseHTML(postText, keywords)

	function parseHTML(htmlString, keywordArr) {
		
		keywordsMatchCount = keywords.map(x => 0)

		for (let i=0; i<keywordArr.length; i++) {
			if (!keywordArr[i]) continue
			if (keywordArr[i].length > 0) {
				htmlString = highlightKeyword(htmlString, keywordArr[i], i)
			}
		}


		return htmlString
		? htmlString.replace('<script', '')
			.split('\n').map(paragraph => (`<p>${paragraph}</p>`))
			.join('')
		: ''
	}

	function highlightKeyword(htmlString, keyword, index) {
		const keywordReplaced = keyword.replace(/\s+/gi, '[^\\w]+')
		const rgx = new RegExp(keywordReplaced, 'gi')
		return htmlString.replace(rgx, function(match) {
			keywordsMatchCount[index]++
			keywordsMatchCount = [...keywordsMatchCount]
			return `<span class='keyword ${ (index === 0) ? 'focus' : '' }'>${ match }</span>`
		})
	}

</script>

<main>
	<h1>SEO Checker</h1>
	<section class='input-fields'>
		<div>
			<label for='focus-keyword'>Focus Keyword</label>
			<input id='focus-keyword' type='text' on:input={e => { keywords[0] = e.target.value }} />
		</div>
		<div>
			<label for='ancillary-keyword'>Ancillary Keywords</label>
			<input id='ancillary-keyword' type='text' on:input={e => { keywords = [ keywords[0], ...e.target.value.split(',').map(str => str.trim())] }} />
		</div>
		<div>
			<label for='post-text'>Post Text</label>
			<textarea id='post-text' on:input={ e => { postText = e.target.value }} />
		</div>
	</section>
	<section class='results'>
		<p><strong>Word Count: </strong>
			{ wordCount }
		</p>
		<table>
			<tbody>
				<tr>
					<th>Keyword</th>
					<th>Count</th>
					<th>Density</th>
				</tr>
				{#each keywords as keyword, i ('keywords-'+i)}
				<tr>
					<td>{ keyword }</td>
					<td class='right-align'>{ keywordsMatchCount[i] }</td>
					<td class='right-align'>{ (keywordsMatchCount[i] / wordCount * 100).toFixed(2) + '%' }</td>
				</tr>
				{/each}
				<tr>
					<th>Total</th>
					<td class='right-align'>{ keywordsMatchCount.reduce((acc, count) => acc + count, 0) }</td>
					<td class='right-align'>{ (keywordsMatchCount.reduce((acc, count) => acc + count, 0) / wordCount * 100).toFixed(2) + '%' }</td>
				</tr>
			</tbody>
		</table>
	</section>
	<section class='post-content'>
		{@html postHTML}
	</section>
</main>

<style>
	main {
		box-sizing: border-box;
		padding: 1em 20vw;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	p {
		max-width: 75ch;
	}

</style>