<script>
	import { browser } from '$app/environment';
	import 'simpledotcss/simple.min.css';
	import socialImage from '$lib/images/social.jpg';

	let english = '';
	let danish = '';
	let greenlandic = '';

	let fromEnglish = true;
	let loading = false;
	/** @type {string[]} */
	let progress = [];

	function switchLanguage() {
		// Clear the input field
		if (fromEnglish) {
			english = '';
		} else if (!fromEnglish) {
			greenlandic = '';
		}
		danish = '';
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
	async function translate(text) {
		loading = true;
		danish = '';
		progress = [
			fromEnglish ? '1/2 Translating English to Danish…' : '1/2 Translating Greenlandic to Danish…'
		];
		try {
			if (fromEnglish) {
				danish = await translateDanish(text);
				progress[0] = '1/2 English to Danish ✓';
				progress.push('2/2 Translating Danish to Greenlandic…');
				greenlandic = await translateGreenlandicHybrid(danish, 'dan2kal');
				progress[1] = '2/2 Danish to Greenlandic ✓';
			} else {
				danish = await translateGreenlandicHybrid(text, 'kal2dan');
				progress[0] = '1/2 Greenlandic to Danish ✓';
				progress.push('2/2 Translating Danish to English…');
				english = await translateDanish(danish);
				progress[1] = '2/2 Danish to English ✓';
			}
		} finally {
			loading = false;
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
		Begin by entering words and simple phrases, then press the <strong>Translate</strong> button. The
		translation moves through Danish as an intermediary; the Danish result and step‑by‑step status messages
		will appear below while the request is processed.
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
			<button on:click={() => translate(fromEnglish ? english : greenlandic)} disabled={loading}>
				Translate
			</button>
			<button class="subtle" on:click={switchLanguage} disabled={loading}>Switch Language</button>
			{#if progress.length}
				<ul class="loading">
					{#each progress as step}
						<li>{step}</li>
					{/each}
				</ul>
			{/if}
		</div>
		<div id="danish">
			<label for="danish-output">dansk (Danish)</label>
			<textarea
				bind:value={danish}
				disabled
				id="danish-output"
				name="Danish"
				rows="5"
				spellcheck="false"
			/>
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
		<button id="copy" on:click={() => copy(fromEnglish ? greenlandic : english)}
			>Copy Result to Clipboard</button
		>
	</section>
	<p>
		This tool works by calling on two services: <a href="https://nutserut.gl/en" target="_blank"
			>Nutserut</a
		>
		(Greenlandic - Danish via its hybrid endpoint) and
		<a href="https://translate.google.ca/?sl=da&tl=en&op=translate" target="_blank"
			>Google Translate</a
		> (Danish - English). The Danish intermediary text and status list above show each stage for transparency.
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
		grid-template-rows: 1fr auto 1fr 1fr auto;
		grid-template-areas:
			'input'
			'controls'
			'danish'
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
	#danish {
		grid-area: danish;
	}
	#english {
		grid-area: var(--english);
	}
	#copy {
		grid-area: copy;
	}
	button.subtle {
		background-color: var(--text);
		color: var(--bg);
		opacity: 0.9;
	}
	.loading {
		margin-left: 1em;
		list-style: none;
		padding-left: 1em;
	}
</style>
