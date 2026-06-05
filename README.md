# DeepThinker / CogniMind

> A digital brain that turns a fleeting idea into systematic, cross-frontier insight.

DeepThinker takes one concept ("the spark") and autonomously researches it across **academia** (papers, preprints, methods) and **industry** (job JDs, products, venture signals), filters for quality, lets **you weight what matters** (human-in-the-loop), and synthesizes the result into a **structured report** — then maps it into a growing knowledge graph.

This repository is a deployable implementation of that flow.

---

## Architecture

```
Browser (index.html, single-page UI)
   │  POST /api/research  { concept, scope }
   ▼
api/research.js  ──►  Claude (claude-sonnet-4-6) + web_search tool  ──►  findings JSON
   │
   │  (you review, weight & approve findings in the UI — human-in-the-loop)
   │
   │  POST /api/report  { concept, approved items + weights }
   ▼
api/report.js    ──►  Claude  ──►  structured Markdown brief
```

- **Frontend** — `index.html`, a self-contained SPA (intro cover → dashboard → inquiry → research → review → report → knowledge map). No build step.
- **`api/research.js`** — calls Claude with the server-side **web search tool** (`web_search_20250305`) to investigate the concept and return structured findings.
- **`api/report.js`** — asks Claude to synthesize the human-approved, weighted findings into a Markdown brief.
- **Stateless** — inquiry state lives in the browser, so it deploys to serverless with zero database.

### Graceful fallback
If no backend is reachable or `ANTHROPIC_API_KEY` is not set, the UI **falls back to built-in sample data**, so it still runs as a working demo (e.g. when you open `index.html` directly as a file).

---

## Deploy to Vercel (≈5 minutes → live URL)

1. Push this folder to a GitHub repo (or use the Vercel CLI).
2. Go to [vercel.com](https://vercel.com) → **Add New… → Project** → import the repo.
3. Framework preset: **Other** (it's static + serverless functions — no build needed).
4. Add an Environment Variable:
   - `ANTHROPIC_API_KEY` = your key from [console.anthropic.com](https://console.anthropic.com)
   - (optional) `DEEPTHINKER_MODEL` = e.g. `claude-opus-4-8`
5. **Deploy.** You'll get a live URL like `https://your-app.vercel.app`.

> Enable web search for your Anthropic organization in the Console if you haven't.

### CLI alternative
```bash
npm i -g vercel
vercel            # follow prompts
vercel env add ANTHROPIC_API_KEY
vercel --prod     # → prints your live URL
```

### Local preview
```bash
npm install
npx vercel dev    # serves the UI + /api functions locally at http://localhost:3000
```
(Or just open `index.html` in a browser to run the offline demo with sample data.)

---

## After deploying: update the promotional materials

The cover, poster and landing pages reference a placeholder app link. Replace it
with your real deployed URL:

```
https://YOUR-APP.vercel.app   →   <your real Vercel URL>
```

Search-and-replace `https://YOUR-APP.vercel.app` across the marketing files.

---

## Notes & limits

- **Cost** — each research run makes one Claude call with web search; each report makes one Claude call. Watch usage on metered keys.
- **Latency** — a research run with web search can take 20–60s. `vercel.json` sets `maxDuration: 60`; very deep runs may need a paid plan for higher limits.
- **Persistence** — this MVP keeps state in the browser. To save inquiries/knowledge base across sessions, add a store (e.g. Vercel Postgres / KV) and persist findings + weights server-side.
- **Models / tools** verified against the Anthropic docs (`claude-sonnet-4-6`, `web_search_20250305`). Bump `DEEPTHINKER_MODEL` as newer models ship.

Built with the Anthropic API. Not affiliated with Anthropic.
