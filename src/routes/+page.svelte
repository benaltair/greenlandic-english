<script>
	import { browser } from '$app/environment';
	import 'simpledotcss/simple.min.css';

	let english = '';
	let greenlandic = '';

	let fromEnglish = true;

	function switchLanguage() {
		// Clear the input field
		if (fromEnglish) {
			english = '';
		} else if (!fromEnglish) {
			greenlandic = '';
		}
		fromEnglish = !fromEnglish;
	}

	/**
	 * @param {string} text
	 */
	function copy(text) {
		if (browser) navigator.clipboard.writeText(text);
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
			.then((data) => parseGoogleTranslate(data[0]));
	}

	/**
	 * @param {any[]} response
	 * @returns {string}
	 */
	function parseGoogleTranslate(response) {
		/**
		 * @type {string[]}
		 */
		let sentences = [];
		response.map((/** @type {string[]} */ i) => {
			if (i[0] !== null) sentences.push(i[0]);
		});
		return sentences.join('');
	}
</script>

<svelte:head>
	<title>Greenlandic to English Translator</title>
	<meta
		name="description"
		content="This tool translates between Greenlandic and English, using Danish as a proxy."
	/>
</svelte:head>

<header>
	<h1>Greenlandic - English Translator</h1>
	<p style="text-align: left;">
		This works by calling on two services: <a href="https://nutserut.gl/en" target="_blank"
			>Nutserut</a
		>
		(Greenlandic - Danish) and
		<a href="https://translate.google.ca/?sl=da&tl=en&op=translate" target="_blank"
			>Google Translate</a
		> (Danish - English). By using Danish as a proxy language, an approximate translation can be provided.
		This is experimental and may break if the external services change.
	</p>
</header>

<main>
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
		<button on:click={() => copy(fromEnglish ? greenlandic : english)}
			>Copy Result to Clipboard</button
		>
	</section>
</main>

<footer>
	This is a simple tool built by Ben Ingham. It is not affiliated with any government or
	organization. You can find the source code on
	<a href="https://github.com/benaltair/greenlandic-english">Github</a>.
</footer>

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
