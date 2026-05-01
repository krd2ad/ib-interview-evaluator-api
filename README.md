# IB Interview Evaluator API

Vercel serverless API that handles AI evaluation for [Investment Banking Interview Prep](https://krd2ad.github.io/ib-interview-rep). Holds the Anthropic key and all evaluation logic — the React frontend never touches the key.

**Endpoint:** `POST https://ib-interview-evaluator-api.vercel.app/api/evaluate`

## Repos

| Repo | Purpose |
|------|---------|
| [`ib-interview-rep`](https://github.com/krd2ad/ib-interview-rep) | React frontend — GitHub Pages |
| [`ib-interview-evaluator-api`](https://github.com/krd2ad/ib-interview-evaluator-api) | Vercel serverless API — this repo |

## Stack

- Vercel serverless functions (TypeScript)
- `@anthropic-ai/sdk` — `claude-haiku-4-5-20251001` with tool use for structured output

## Architecture

```
React frontend (GitHub Pages)
  → POST /api/evaluate
      headers: { x-site-password: <password>, Content-Type: application/json }
      body: { question, answer, category, difficulty }
  → Vercel validates x-site-password against SITE_PASSWORD env var
  → calls Anthropic with system prompt + rubric
  → returns { feedback: EvaluationResponse, costUsd: number }
```

## Request / response shape

```ts
// Request body
type EvaluationRequest = {
  question: string
  answer: string
  category: string
  difficulty?: string
}

// Response body
type EvaluationResult = {
  feedback: {
    score: number                   // 1–10
    oneSentenceSummary: string
    whatWentWell: string[]
    whatToImprove: string[]
    suggestedImprovedAnswer: string
    recommendation: "try_again" | "move_on"   // move_on when score >= 8
  }
  costUsd: number
}
```

## Environment variables (set in Vercel)

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Anthropic API key (`sk-ant-...`) |
| `SITE_PASSWORD` | Shared password checked against `x-site-password` header |

## Dev

```bash
npm install
npm run dev      # vercel dev — runs the function locally on http://localhost:3000
```

## Deployment

Push to `main` — Vercel auto-deploys. To change the model or evaluation prompt, edit `api/evaluate.ts`.
