---
title: 'NLP.js NLU Integration'
excerpt: 'Turn raw text into structured meaning with the Jovo Framework integration for the open source natural language understanding service NLP.js.'
---

# NLP.js NLU Integration

Turn raw text into structured meaning with the Jovo Framework integration for the open source natural language understanding service NLP.js.

## Introduction

[NLP.js](https://github.com/axa-group/nlp.js) is an open source [natural language understanding (NLU)](https://www.jovo.tech/docs/nlu) library with features like entity extraction, sentiment analysis, and language detection.

Since it is an open source service, you can host NLP.js on your own servers without any external API calls.

You can use the Jovo NLP.js integration for projects where you receive raw text input that needs to be translated into structured meaning to work with the Jovo intent structure. Learn more in the [NLU integration docs](https://www.jovo.tech/docs/nlu).

Smaller NLP.js language models are fast to train and can even be used on serverless infrastructure like [AWS Lambda](https://www.jovo.tech/marketplace/server-lambda) without having to use any additional server infrastructure. We recommend taking a close look at the execution times though, as larger models can take quite some time to build.

## Installation

You can install the plugin like this:

```sh
$ npm install @jovotech/nlu-nlpjs
```

NLU plugins can be added to Jovo platform integrations. Here is an example how it can be added to the [Jovo Core Platform](https://www.jovo.tech/marketplace/server-lambda) in your `app.ts` [app configuration](https://www.jovo.tech/marketplace/platform-core):

```typescript
import { CorePlatform } from '@jovotech/platform-core';
import { NlpjsNlu } from '@jovotech/nlu-nlpjs';

// ...

app.configure({
  plugins: [
    new CorePlatform({
      plugins: [new NlpjsNlu()],
    }),
    // ...
  ],
});
```

## Configuration

The following configurations can be added:

```typescript
new NlpjsNlu({
  languageMap: { /* ... */ },
  preTrainedModelFilePath: './model.nlp',
  useModel: false,
  modelsPath: './models',
  setupModelCallback: (platform: Platform, nlpjs) => { /* ... */ },
}),
```

- `languageMap`: Maps locales to NLP.js language packages. The default is an empty object. See [language configuration](#language-configuration) for more information.
- `useModel` and `preTrainedModelFilePath`: NLP.js can take a pre-trained model if `useModel` is set to `true`. It looks for the model in the `preTrainedModelFilePath`. The default is `./model.nlp`.
- `setupModelCallback`: A function that can be passed to set up NLP.js. The first parameter is the current `Platform` and the second parameter is the `Nlp`-instance of NLP.js.

Depending on the configuration, NlpjsNlu will try to use the `setupModelCallback` if it exists.
Otherwise, the integration will check if `useModel` is set to `true`, if that's the case, the model is getting loaded from `preTrainedModelFilePath`.
If `setupModelCallback` does not exist and `useModel` is falsy, the integration will attempt to build a model based on the local models and train it.

### Language Configuration

For each language you want to use with NLP.js, you need to download the respective language package. Here is an example for [`en` (English)](https://github.com/axa-group/nlp.js/tree/master/packages/lang-en):

```sh
$ npm install @nlpjs/lang-en
```

You can then add this to your `languageMap`:

```typescript
import { LangEn } from '@nlpjs/lang-en';
// ...

new NlpjsNlu({
  languageMap: {
    en: LangEn,
    // ...
  },
  // ...
}),
```

Each key (in the above case `en`) represents a locale that can be found in your [Jovo Model](#jovo-model).

You can find more information about [supported languages](https://github.com/axa-group/nlp.js/blob/master/docs/v4/language-support.md) and [available packages](https://github.com/axa-group/nlp.js/tree/master/packages) in the official NLP.js docs.

## Entities

You can access NLP.js entities by using the `$entities` property. You can learn more in the [Jovo Model](https://www.jovo.tech/docs/models) and the [`$entities` documentation](https://www.jovo.tech/docs/entities).

The NLP.js entity values are translated into the following Jovo entity properties:

```typescript
{
  value: utteranceText, // what the user said
  resolved: option, // the resolved value
  id: option, // same as resolved, since NLP.js doesn't support IDs
  native: { /* raw API response for this entity */ }
}
```

## Jovo Model

You can use the [Jovo Model](https://www.jovo.tech/docs/models) to turn the language model files in your `models` folder into an NLP.js model. [Learn more about the NLP.js Jovo Model integration here](https://www.jovo.tech/marketplace/nlu-nlpjs/model).


## Limitations

NLP.js does not support multiple entities of the same type in the same phrase, for example `fly from {fromCity} to {toCity}`. In that example, it is possible that the input `fly from Berlin to Amsterdam` is mapped to two `toCity` entities.

If you need to use a pattern like this, we recommend taking a look at other NLU integrations like [Snips NLU](https://www.jovo.tech/marketplace/nlu-snips).