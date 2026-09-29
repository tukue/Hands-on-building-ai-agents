---
title: Public Data Investment Research
emoji: 📊
colorFrom: blue
colorTo: green
sdk: gradio
app_file: app.py
python_version: '3.11'
pinned: false
---

# Public-data Investment Research Agent — RAG in Colab

A portfolio MVP that researches company filings and produces cited, qualitative
research views for human review. No trading, personal suitability assessment or
price forecasting. Built around public SEC data, not private account information.

## Actual data included and market tools

The source bundle contains **six actual Apple financial facts**, fetched from the
SEC companyfacts API, for the FY2025 filing (accession 0000320193-25-000079).
`samples/actual_sec_facts.json` preserves raw metric IDs, values, units, periods,
filing date and the API source. `actual_sec_sample.json` renders those structured
facts into searchable passages. They are not invented data and not verbatim
report excerpts. This is a small archived sample, not complete company research.

A recorded AAPL response from Twelve Data's public `apikey=demo` endpoint is
included separately. It is a provider demo response, not proof of live-feed access.
Its capture time and provider timestamp remain visible. It is never labeled live.

`advisor.py` combines the RAG agent with optional market tools in `market_data.py`:
- Quote fetched on request, with a 30-second raw-response cache.
- Symbol/exchange/currency checks for AAPL, MSFT or IBM.
- Provider quote timestamp, retrieval time, market state and feed labels.
- Optional historical daily prices and deterministic risk calculations.
- Market evidence is supplied to the bounded LLM retrieval loop with citation IDs.
The quote/history calls are controlled by user options; the LLM chooses document
searches. There is no WebSocket stream or autonomous portfolio/trading loop.

Set `TWELVE_DATA_API_KEY` as a secret. Set `MARKET_DATA_MODE` only after verifying
your account's entitlement: `realtime`, `delayed`, `end_of_day`, or `unknown`.
This configuration is an operator declaration; the client cannot verify licensing.
A recent quote timestamp is not sufficient evidence of a real-time subscription.
During an open market, unknown entitlement, missing timestamps, unknown market
state or a stale real-time quote block price-aware assessment. When closed, the
last quote is explicitly labeled as such. Never claim it is a current executable
bid or ask. Real-time means snapshot-on-request here, not streaming.

For fresh API mode, first ingest a full public filing corpus. The bundled sample
cannot be combined with fresh prices to imply a complete current recommendation.
`include_history=True` requests adjusted daily data, excludes UTC-current-day bars
and reports the price-return window, annualized daily volatility and drawdown.
Corporate-action adjustments depend on provider conventions; these are not a
validated total-return series. The final answer still requires human review.
Check the provider's exchange coverage, quota and redistribution rights before
public display. Do not put market API keys in URLs printed in logs or notebooks.

## Start in Colab

1. Upload `Investment_Research_Agent_Colab.ipynb` to https://colab.research.google.com/.
2. Select a Python CPU runtime and run the project-generation and install cells.
3. Run the bundled actual SEC sample and tests. This needs no API keys or LLM.
4. Set `SEC_USER_AGENT` through Colab Secrets to `YourApp/1.0 YourName your-real-email`.
5. Set `FETCH_PUBLIC=True` in the ingestion cell, select IBM/MSFT/AAPL, then run it.
6. Build the index and inspect the passages before enabling AI.
7. Add `HF_TOKEN` (Inference Providers permission), choose a supported `HF_MODEL`
   in https://huggingface.co/playground, and set `USE_LLM=True`.
8. Launch Gradio. Select a company present in your corpus and approve the request.

Do not run every optional cell blindly. Ingestion, semantic model download,
LLM calls, publishing and UI launch have separate controls. Colab runtime files
are temporary; download source before disconnecting. Source regeneration does not
overwrite files unless `OVERWRITE_PROJECT=True`.

## Included implementation

- SEC submissions API discovery and bounded primary-filing HTML downloads.
- One to three configured companies: IBM, Microsoft and Apple.
- Latest 10-K and 10-Q found in recent submissions, plus latest subsequent amendment
  for each type when available. No historical-submissions traversal, exhibits,
  8-K, international forms, news, or exhaustive amendment resolution yet.
- Company/CIK check, source URL, accession, publication/retrieval dates and hashes.
- HTML text extraction and overlapping chunks with stable citation IDs.
- TF-IDF vector retrieval by default; optional sentence-transformer semantic retrieval.
- A bounded JSON-action LLM loop: up to two searches, then a cited answer.
- Exact-quote/citation validation, synthetic-data labels and stale-snapshot blocking.
- Gradio UI, offline tests, separate original price-risk module, and exportable source.
- Tested-code deployment workflow and daily public-data workflow.

TF-IDF is a lexical RAG baseline, not a neural embedding model. Semantic mode uses
`sentence-transformers/all-MiniLM-L6-v2` on CPU and downloads model weights; it needs
`requirements-semantic.txt`. The index is rebuilt from text on startup/reload.
Neither backend is a persistent vector database. Similarity thresholds are starter
heuristics, not calibrated relevance probabilities. Benchmark both on real filings.
Small character chunks may split sentences/tables; improve section-aware chunking
before making fine-grained accounting comparisons. The embedding model has a token
limit, so inspect truncation when adapting the chunk size.

## Source and numerical limitations

SEC APIs are public without an API key; identify automated requests and comply
with fair access. This client runs sequentially with at least 0.3 seconds between
request starts. On errors it stops rather than bypassing restrictions. Some cloud
IPs may receive 403 responses; do not disguise traffic or repeatedly retry.

Public filings contain issuer statements, not independent verification. Public
access does not automatically grant unrestricted republication. Keep the dataset
private while assessing reuse rights. Publication dates are not reporting-period
end dates. Financial tables are flattened by this text extractor; use structured
XBRL and deterministic code for precise numerical comparisons in a later version.

The original `agent.py` remains available for optional Alpha Vantage unadjusted
price calculations. It is separate from the RAG agent; those numbers are not
silently combined into its recommendations. The unified advisor can additionally cite Twelve Data quote/history evidence.

## Answer format

- Research view: favourable, mixed, cautious, or insufficient_evidence.
- Typed statements: findings, risks, interpretations, next checks.
- Exact supporting excerpt and a resolvable citation ID for each statement.
- Source URL, publication date, publication age, retrieval time and snapshot time.
- Action trace and AWAITING_HUMAN_REVIEW status.

A quote matching a source does **not** prove the statement is supported by it.
Entailment and balanced interpretation require evaluation and human review.
Prompts treat sources as untrusted input; this reduces risk but is not a complete
prompt-injection defense. No code execution or arbitrary URL tool is exposed.
Favourable views are not calibrated predictions or personalized buy signals.
A daily download does not make an old filing new; inspect publication dates.

## GitHub → Hugging Face setup

Use a separate investment-agent repository. Do not overwrite the DevSecOps project.
Download the source ZIP from Colab, unzip it, and preserve `.github/workflows/`.
Create an empty GitHub repo, then:

```bash
cd investment-agent
git init -b main
git add .
git commit -m "Add public-data investment RAG agent"
git remote add origin https://github.com/YOUR_USERNAME/YOUR_INVESTMENT_REPO.git
git push -u origin main
```

If your repo already has history, clone it first and copy source into that checkout.
Use a credential manager, not credentials embedded in remote URLs.
Create a **private Gradio Space** and a **private dataset repository** on Hugging Face.
Add these GitHub repository settings:

| Name | Kind | Purpose |
|---|---|---|
| HF_TOKEN | Secret | Write permission for the destination Space |
| SEC_USER_AGENT | Secret | App identity and real contact email for SEC requests |
| HF_DATASET_WRITE_TOKEN | Secret | Write permission for the corpus dataset |
| HF_SPACE_ID | Variable | Your username/space-name |
| HF_DATASET_ID | Variable | Your username/dataset-name |
| WATCHLIST | Variable | IBM initially; later IBM,MSFT,AAPL |

The deploy workflow runs tests before upload on main. Pull requests run tests only.
The official hub-sync action mirrors files; manage app files on GitHub, not through
separate edits on Spaces. Review deletion behavior before pointing at an existing Space.
The data refresh workflow runs at 05:17 UTC daily (07:17 Stockholm summer, 06:17 winter)
and supports manual dispatch. Scheduling can be delayed; it is not an exact-time SLA.
Monitor failed runs and scheduled-workflow inactivity policies in GitHub.
No workflow is active until you configure and push it to your repository.

Set these **Space runtime** values separately:

| Name | Type | Purpose |
|---|---|---|
| HF_TOKEN | Secret | Inference Providers permission |
| HF_DATA_TOKEN | Secret | Read permission for the private dataset |
| HF_MODEL | Variable | Currently available chat model ID |
| TWELVE_DATA_API_KEY | Secret | Market quote/history API key |
| MARKET_DATA_MODE | Variable | Verified account feed mode; defaults to unknown |
| HF_DATASET_ID | Variable | Same dataset repo as the refresh job |
| RETRIEVAL_BACKEND | Variable | tfidf initially |

A CPU Space is sufficient with hosted generation. Inference can incur charges.
Keep it private while using owner-funded keys. Public launch needs authentication,
per-user quotas, budget controls and privacy review; Gradio queue limits alone
are not adequate. Select AAPL for the bundled actual sample; choose a freshly ingested company for API mode.

## How daily updates reach the app

The job downloads the selected filing set, hashes documents, validates full company
coverage, and writes a candidate snapshot atomically. It currently refetches and
rebuilds a small corpus; content hashes support deduplication/integrity but there is
no incremental embedding cache yet. All configured companies must succeed.

One Hub upload commits the complete JSON snapshot. The app checks for new dataset
revisions on requests, at most every five minutes, downloads an exact commit, builds
a new index, and replaces its in-memory reference after validation. It retains the
last good in-memory index if refresh fails. On a cold start with no usable dataset
it reports an error; it does not silently replace configured public data with demo.
Snapshots older than 48 hours block current research responses.
A new Space worker starts with its own cache; no shared durable cache is claimed.

## Evaluation and next work

39 offline tests cover calculations, retrieval, source filtering, fabricated quotes,
agent limits, HTML extraction, filing selection and failed-update preservation.
The notebook also measures Recall@3 on a tiny synthetic set. That is a smoke test,
not a real-financial-data accuracy benchmark. Add at least 20–50 manually reviewed
filing questions, negative questions and adversarial source passages before citing
quality numbers in your portfolio. Evaluate precision, claim support, abstention,
latency and API cost. Test semantic retrieval separately before enabling it.

Actual sample facts and the recorded demo quote were retrieved successfully from public APIs. Full-filing ingestion, entitled live quotes, hosted inference, semantic weight downloads and deployment still require your configuration and were not validated end-to-end. Dependency ranges should be locked after validating
your clean Colab/Space runtime. No external account was modified during creation.

## Official references checked for this version

- SEC APIs: https://www.sec.gov/search-filings/edgar-application-programming-interfaces
- SEC fair access: https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data
- Inference: https://huggingface.co/docs/huggingface_hub/en/guides/inference
- Space sync: https://huggingface.co/docs/hub/spaces-github-actions
- Hub downloads: https://huggingface.co/docs/huggingface_hub/en/guides/download
- GitHub schedules: https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows

- Twelve Data quote/history documentation: https://twelvedata.com/docs
