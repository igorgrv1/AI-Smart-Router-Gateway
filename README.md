# AI Smart Router Gateway

A lightweight Fastify gateway that routes chat requests through OpenRouter and lets the app choose the best available model strategy for the job. The idea is simple: keep one API, plug in multiple models, and tune routing based on prompt characteristics, cost, latency, or throughput.

This project is a good starting point when you want to optimize token spend without hard-coding a single model for every request.

## Why this project

Large language models are not all equally good for every prompt. Some requests need:

- maximum speed
- the cheapest available option
- higher quality reasoning
- better throughput for many concurrent queries

Instead of picking one model forever, this app sends the request through OpenRouter and chooses a model strategy from a configured pool.

The router can be configured to prefer:

- price
- throughput
- latency

and the model selection is driven by the OpenRouter provider configuration.

## What it does

This project exposes a single HTTP endpoint:

- POST /chat

It accepts a prompt payload and forwards it to OpenRouter using a list of configured models. The response includes the selected model and the generated content.

## Example response

This is the kind of result the endpoint returns:

```json
{
  "model": "nvidia/nemotron-3-nano-30b-a3b:free",
  "content": "**Rate limiting** is a technique for controlling how many requests or actions a user, client, or system can perform within a given time window. It limits the rate."
}
```

The screenshot in the repository shows the live response from the local server.

## Project structure

```text
src/
├── config.ts            # model list, provider sorting, temperature, token limits
├── openrouterService.ts # OpenRouter client and request generation
├── server.ts            # Fastify server and /chat route
├── index.ts             # app bootstrap

tests/
└── router.e2e.test.ts   # smoke tests for model routing behavior
```

## Getting started

### 1) Configure your OpenRouter account

Follow these steps to get a working API key and test the models in OpenRouter:

1. Go to OpenRouter: https://openrouter.ai/workspaces/default?utm_source=signup-success
2. Create an account or sign in.
3. Get your API key from the OpenRouter dashboard.
4. Explore free text models here: https://openrouter.ai/models?input_modalities=text&max_price=0
5. In the model list, select only text models and filter by free options.
6. Use the activity page to monitor requests and usage: https://openrouter.ai/activity
7. Manage your keys in the workspace: https://openrouter.ai/workspaces/default/keys
8. You can set a monthly spending limit in dollars for each API key.
9. Test models directly in the OpenRouter chat UI: https://openrouter.ai/chat
10. You can try more than one model at the same time to compare responses and choose the best fit.

### OpenRouter checklist (em português)

1. Acesse o OpenRouter: https://openrouter.ai/workspaces/default?utm_source=signup-success
2. Pegue sua API KEY.
3. Vá para: https://openrouter.ai/models?input_modalities=text&max_price=0
4. Selecione o tipo texto e gratuito.
5. Use as API keys em: https://openrouter.ai/activity
6. Gerencie elas em: https://openrouter.ai/workspaces/default/keys
7. Você pode selecionar o limite em dólares que a API key pode gastar.
8. Teste o modelo em: https://openrouter.ai/chat
9. Selecione o modelo que você quer testar e use!
10. Você pode selecionar mais de um modelo para testar diferentes respostas.

### 2) Install dependencies

```bash
npm install
```

### 3) Create a local environment file

```bash
cp .env.example .env
```

### 4) Add your OpenRouter API key

```env
OPENROUTER_API_KEY=your_api_key_here
```

### 5) Start the app

```bash
npm run dev
```

The server listens on port 3000 by default.

## Using the API

Send a chat request:

```bash
curl -X POST http://localhost:3000/chat \
  -H "Content-Type: application/json" \
  -d '{"question":"What is rate limiting?"}'
```

Example response:

```json
{
  "model": "nvidia/nemotron-3-nano-30b-a3b:free",
  "content": "Rate limiting is a technique used to control how many requests or actions can be made in a given time period."
}
```

## Configuration

The router configuration lives in [src/config.ts](src/config.ts). You can adjust:

- `models`: the model list provided to OpenRouter
- `provider.sort.by`: one of `price`, `throughput`, or `latency`
- `temperature`: creativity level
- `maxTokens`: maximum output length
- `systemPrompt`: default system instructions

Example:

```ts
export const config = {
  apiKey: process.env.OPENROUTER_API_KEY!,
  httpReferer: 'http://pos-ia.com',
  xTitle: 'SmartModelRouterGateway',
  port: 3000,
  models: [
    'arcee-ai/trinity-large-preview:free',
    'nvidia/nemotron-3-nano-30b-a3b:free',
  ],
  provider: {
    sort: {
      by: 'throughput',
      partition: 'none'
    }
  }
}
```

## Routing strategy

The current implementation is intentionally simple and flexible:

- one API endpoint
- multiple provider/model choices
- a configurable optimization strategy
- OpenRouter handles the actual provider/model selection and routing logic

This makes it easy to extend later with smarter prompt-based logic, such as:

- short/simple prompts -> cheaper, faster models
- complex reasoning prompts -> higher-quality models
- code generation -> model families best suited for code
- long context prompts -> larger context models

## Development

Run tests:

```bash
npm test
```

The repository includes a small end-to-end test suite that validates the selected model routing behavior.

## License

This project is licensed under the ISC License.

## Notes

This repo is a clean, practical example of building a gateway around OpenRouter. It keeps the integration lightweight, but the architecture scales well if you want to add prompt classifiers, cost-aware policies, fallbacks, logging, or a production-ready API layer.
