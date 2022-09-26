<script>
	import 'simpledotcss';

	let english = '';
	let greenlandic = '';

	let fromEnglish = true;

	function switchLanguage() {
		// Clear the input field
		// if (fromEnglish) {
		// 	english = '';
		// } else if (!fromEnglish) {
		// 	greenlandic = '';
		// }
		fromEnglish = !fromEnglish;
	}

	/**
	 * @param {string} text
	 */
	function translate(text) {
		if (fromEnglish) {
			translateDanish(text).then((r) => translateGreenlandic(r).then((r) => (greenlandic = r)));
		} else if (!fromEnglish) {
			translateGreenlandic(text).then((r) => translateDanish(r).then((r) => (english = r)));
		}
	}

	/**
	 * @param {string} text
	 */
	function translateGreenlandic(text) {
		return fetch(
			`https://nutserut.gl/callback.php?a=${fromEnglish ? 'dan2kal' : 'kal2qdx'}&t=${encodeURI(
				text
			)}`
		)
			.then((r) => r.json())
			.then((t) => {
				if (fromEnglish) {
					return t?.output;
				} else if (!fromEnglish) {
					return t?.moved;
				}
			});
	}

	/**
	 * @param {string} text
	 */
	function translateDanish(text) {
		return fetch(
			`https://translate.googleapis.com/translate_a/single?client=gtx&sl=${
				fromEnglish ? 'en' : 'da'
			}&tl=${fromEnglish ? 'da' : 'en'}&dt=t&q=${text}`
		)
			.then((response) => response.json())
			.then((data) => data[0][0][0]);
	}
</script>

<h1>Greenlandic - English Translator</h1>
<p>This works by calling on two services:</p>
<ul>
	<li><a href="https://nutserut.gl/en">Nutserut</a> (Greenlandic - Danish)</li>
	<li>
		<a href="https://translate.google.ca/?sl=da&tl=en&op=translate">Google Translate</a> (Danish - English)
	</li>
</ul>
<p>This is experimental and may break if the external services change.</p>

<section
	style="--english:{fromEnglish ? 'input' : 'result'}; --greenlandic:{fromEnglish
		? 'result'
		: 'input'};"
>
	<div id="greenlandic">
		<label for="greenlandic">Greenlandic</label>
		<textarea bind:value={greenlandic} disabled={fromEnglish} name="Greenlandic" rows="10" />
	</div>
	<div id="controls">
		<button on:click={() => translate(fromEnglish ? english : greenlandic)}>Translate</button>
		<button on:click={switchLanguage}>Switch Language</button>
	</div>
	<div id="english">
		<label for="english">English</label>
		<textarea bind:value={english} disabled={!fromEnglish} name="English" rows="10" />
	</div>
	<button disabled>Copy Result to Clipboard</button>
</section>

<style>
	section {
		display: grid;
		grid-template-columns: 1fr;
		grid-template-rows: 1fr auto 1fr;
		grid-template-areas:
			'input'
			'controls'
			'result'
			'copy';
	}
	#controls {
		grid-area: controls;
	}
	#greenlandic {
		grid-area: var(--greenlandic);
	}
	#english {
		grid-area: var(--english);
	}
</style>
