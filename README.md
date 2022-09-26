# English - Greenlandic Translator

This is a simple translator that translates English to Greenlandic and Greenlandic to English. It pulls directly from [Nutserut](https://nutserut.gl/en), a service from [The Language Secretariat of Greenland](https://oqaasileriffik.gl/en/). Using Danish as a proxy language, it then pulls from the [Google Translate API](https://cloud.google.com/translate/docs/) to deliver an English translation, and vice versa.

To distribute this service widely, consultation would be needed with the Language Secretariat of Greenland to ensure that their server is not abused. This is a proof of concept, and is not intended to be used in production.

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of this app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
