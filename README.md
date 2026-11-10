# Pontis — Latin language companion

Pontis is a Next.js app with a split Dictionary / Translator workspace. Dictionary lookups and inflection tables are provided by the public `latindictionary.io` API. Sentence translations are drafted by OpenAI from the dictionary evidence returned for each token.

## Model choice

Default: **`gpt-6-luna`**, with **low** reasoning effort. OpenAI positions GPT-6 Luna for focused, cost-sensitive, high-volume tasks. It supports the Responses API and structured outputs. The model is configurable with `OPENAI_MODEL` and `OPENAI_REASONING_EFFORT`.

The default model is intentionally economical. A sample request using approximately 1,000 input tokens and 300 output tokens would cost about $0.00025 at the listed standard rates ($0.10 / million input tokens and $0.50 / million output tokens). Actual cost varies with passage length, dictionary evidence, prompt overhead and output size. Have the account owner set a hard project spending limit in the OpenAI Platform; a billing limit is not a substitute for application-side abuse controls. OpenAI notes that hard-limit enforcement is not instantaneous, so slight overrun is possible. A monthly spend notification/soft limit alone does not stop requests. Keep the key private and monitor usage regularly.

## Requirements

- Node.js 20.9 or later (Node 22 recommended)
- npm
- An OpenAI API key for sentence translation
- No API key is required for the public latindictionary.io dictionary endpoints

## Run locally

1. Extract this project and open its folder in Terminal or VS Code.
2. Install dependencies:

   ```bash
   npm install
   ```

3. Copy `.env.example` to `.env.local` and add your OpenAI key:

   ```env
   OPENAI_API_KEY=your_secret_key
   OPENAI_MODEL=gpt-6-luna
   OPENAI_REASONING_EFFORT=low
   ```

4. Start the development server:

   ```bash
   npm run dev
   ```

5. Open `http://localhost:3000`.

Never prefix the key with `NEXT_PUBLIC_`. Do not commit `.env.local` or share the secret in chat.

## Deploy to Vercel

1. Push the project to a private GitHub repository.
2. Import that repository into Vercel.
3. In Vercel → Project → Settings → Environment Variables, add `OPENAI_API_KEY`, `OPENAI_MODEL`, and `OPENAI_REASONING_EFFORT` for Production (and Preview if needed).
4. Deploy or redeploy after adding/changing environment variables.

Vercel automatically detects Next.js. No special build command is needed; the build script is `next build`.

## API routes

- `GET /api/word/{word}` — one-word Latin lookup.
- `GET /api/inflections/{lemma}` — conjugation / declension tables.
- `POST /api/translate` — accepts `{ "text": "..." }`; performs dictionary lookup for each token, then returns a structured draft translation. A single word is routed to dictionary lookup and does not invoke AI.

## Data flow

```text
Browser
  ├─ GET /api/word/{word}
  │    └─ Server → latindictionary.io /la-to-en/{word}
  ├─ GET /api/inflections/{lemma}
  │    └─ Server → latindictionary.io /inflection-table?lemma={lemma}
  └─ POST /api/translate { text }
       ├─ Server tokenises passage and deduplicates lookups
       ├─ Server → latindictionary.io for each distinct token
       ├─ Server assembles compact dictionary evidence
       ├─ Server → OpenAI Responses API (gpt-6-luna by default)
       ├─ Structured JSON is validated and word meanings are re-grounded
       └─ Browser receives draft translation, glosses, grammar notes, ambiguities
```

## Caching and limits

- Dictionary GET requests use Next.js's Data Cache for one hour.
- Successful sentence translations use Next.js's Data Cache for one hour; the cache key includes the original passage, dictionary evidence and model.
- Passage length is capped at 1,500 characters, 90 word tokens and 45 distinct words to bound upstream requests and prompt size.
- The public dictionary API documents a limit of 600 requests/minute for ordinary endpoints per source IP.
- This first version does not include accounts, persistent user quotas, Turnstile, or a global daily AI cap. For a public launch, add persistent rate limiting (e.g. Vercel KV / Upstash), a daily global cap, and an anti-abuse challenge before opening access widely. This starter does not yet enforce those protections; do not expose the translation endpoint publicly until they are added.

## Grounding behaviour

The API provides headwords, definitions, parts of speech, frequency and morphology when available. Pontis passes this information to the model as evidence. After the model returns, the server replaces each word gloss's lemma, part of speech and English meaning with the dictionary response. Words without a dictionary hit remain unresolved; if no words in a passage are found, the model is not called.

The sentence itself is still an AI-assisted draft. The dictionary API's available parse and morphology depend on the data returned for each entry; Pontis does not claim that every Latin form has a unique parse.

## Useful commands

```bash
npm run dev
npm run typecheck
npm test
npm run build
```
# Pontis
