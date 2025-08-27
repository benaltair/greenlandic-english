<script>
	import { browser } from '$app/environment';
	import 'simpledotcss/simple.min.css';
	import socialImage from '$lib/images/social.jpg';

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
			translateDanish(text).then((r) =>
				translateGreenlandicHybrid(r, 'dan2kal').then((r) => (greenlandic = r))
			);
		} else {
			translateGreenlandicHybrid(text, 'kal2dan').then((r) =>
				translateDanish(r).then((r) => (english = r))
			);
		}
	}

	/**
	 * Legacy Nutserut translation.
	 * @param {string} text
	 * @param {'kal2dan' | 'dan2kal'} direction
	 */
	// eslint-disable-next-line no-unused-vars
	function translateGreenlandicClassic(text, direction) {
		const action = direction === 'dan2kal' ? 'dan2kal' : 'kal2qdx';
		return fetch(`https://nutserut.gl/callback.php?a=${action}&t=${encodeURIComponent(text)}`)
			.then((r) => r.json())
			.then((t) => {
				if (direction === 'dan2kal') {
					let withoutBreaks = t?.output?.replace(/\n\n\n\n\n/gm, '');
					return withoutBreaks;
				} else {
					return t?.moved ?? t?.output;
				}
			});
	}

	/**
	 * Hybrid Nutserut translation.
	 * @param {string} text
	 * @param {'kal2dan' | 'dan2kal'} direction
	 */
	function translateGreenlandicHybrid(text, direction) {
		const action = direction === 'dan2kal' ? 'm-dan2kal' : 'h-kal2dan';
		return fetch(`https://nutserut.gl/callback.php?a=${action}&t=${encodeURIComponent(text)}`)
			.then((r) => r.json())
			.then((t) => t?.output);
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
	<meta name="twitter:card" content="summary_large_image" />
	<meta name="og:image" content={socialImage} />
</svelte:head>

<header>
	<svg xmlns="http://www.w3.org/2000/svg" height="48" width="48"
		><path
			d="m23.75 44 9.05-24h4.1l9.3 24h-4.35l-2.05-6.3h-9.6L27.85 44Zm7.55-9.8h7.1l-3.5-9.7h-.1ZM8 38l-2.75-2.75 10.2-10.2q-1.9-2.2-3.375-4.425Q10.6 18.4 9.5 16h4.35q.85 1.65 1.875 3.125t2.325 3.025q2.25-2.35 3.75-4.875T24.35 12H2V8h14V4h4v4h14v4h-5.65q-1.1 3.45-2.925 6.775Q23.6 22.1 20.95 25.1l4.9 4.95-1.5 4.05L18 28Z"
		/></svg
	>
	<h1>Greenlandic - English Translator</h1>
	<p>
		Begin by entering words and simple phrases, then press the <strong>Translate</strong> button.
	</p>
</header>

<main>
	<section
		class:fromEnglish
		style="--english:{fromEnglish ? 'input' : 'result'}; --greenlandic:{fromEnglish
			? 'result'
			: 'input'};"
	>
		<div id="greenlandic">
			<label for="greenlandic-input">kalaallisut (Greenlandic)</label>
			<textarea
				bind:value={greenlandic}
				disabled={fromEnglish}
				id="greenlandic-input"
				name="Greenlandic"
				rows="5"
				spellcheck="false"
			/>
		</div>
		<div id="controls">
			<button on:click={() => translate(fromEnglish ? english : greenlandic)}>Translate</button>
			<button class="subtle" on:click={switchLanguage}>Switch Language</button>
		</div>
		<div id="english">
			<label for="english-input">English</label>
			<textarea
				bind:value={english}
				disabled={!fromEnglish}
				id="english-input"
				name="English"
				rows="5"
			/>
		</div>
		<button on:click={() => copy(fromEnglish ? greenlandic : english)}
			>Copy Result to Clipboard</button
		>
	</section>
	<p>
		This tool works by calling on two services: <a href="https://nutserut.gl/en" target="_blank"
			>Nutserut</a
		>
		(Greenlandic - Danish) and
		<a href="https://translate.google.ca/?sl=da&tl=en&op=translate" target="_blank"
			>Google Translate</a
		> (Danish - English). By using Danish as a proxy language, an approximate translation can be provided.
		This is experimental and may break if the external services change. It currently works best with
		words and simple phrases, as the underlying translation service is still being developed. Specifically,
		this tool translates between English and West Greenlandic (kalaallisut, the official language of
		Greenland) and vice versa.
	</p>
</main>

<footer>
	This is a simple tool built by Ben Ingham in September 2022. It is not affiliated with any
	government or organization. You can find the source code on
	<a href="https://github.com/benaltair/greenlandic-english">Github</a>.
</footer>

<style>
	svg {
		fill: var(--text);
		margin-top: 3em;
	}
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
	section.fromEnglish {
		--greenlandic: result;
		--english: input;
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
	button.subtle {
		background-color: var(--text);
		color: var(--bg);
		opacity: 0.9;
	}
</style>
