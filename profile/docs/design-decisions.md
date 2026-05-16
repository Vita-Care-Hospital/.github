# Design decisions & trade-offs
## 1. Hand-written agent loop vs LangChain “agents”

**Choice:** Keep `run_chat` as an explicit loop (LLM → tools → LLM) and use LangChain only for document loaders, splitting, and Qdrant vector store glue.

**Trade-off:** More bespoke code to maintain, but the control flow stays readable in one place and is easy to step through in a debugger or interview.

**With more time:** Extract a tiny internal “tool runtime” interface (typed tool results + retries) and add contract tests against recorded LLM transcripts—without adopting a full agent framework.

---

## 2. Groq (hosted) vs Ollama (local) as the default path

**Choice:** Prefer Groq for demos (latency, throughput) with Ollama as a configurable fallback for offline and embedding workloads.

**Trade-off:** Hosted keys, provider limits, and less control over model weights; local Ollama shifts complexity to the operator’s machine (RAM, GPU/CPU, model pulls).

**With more time:** Add a small model-router layer (task → model) and structured logging of token usage and latency per request.

---

## 3. SQLite file store vs in-memory bookings

**Choice:** `hospital-api` persists slots and bookings in a local **SQLite** file (`data/hospital.db` by default) with WAL mode for simple concurrency.

**Trade-off:** Demo operators must delete the DB file (or clear tables) to fully reset slot calendars; schema migrations are manual for this toy service.

**With more time:** Managed Postgres, Alembic migrations, and idempotent booking APIs (idempotency keys, row-level locking on slots).

---

## 4. Regex-based “emergency” gate vs clinical triage systems

**Choice:** A lightweight `EMERGENCY_PATTERNS` check before calling the LLM, with a fixed escalation message—**not** a medical device and not a substitute for real triage.

**Trade-off:** False negatives/positives; regex does not understand context or severity.

**With more time:** Route crisis language to a static hotline UI and remove the LLM from that path entirely; partner with clinical stakeholders for real requirements.

---

## 5. Qdrant + local embeddings vs a hosted vector + embedding API

**Choice:** Qdrant in Docker and Ollama `nomic-embed-text` for embeddings to keep the demo runnable without extra paid services.

**Trade-off:** Operational overhead (another container); embedding throughput tied to local hardware; cold-start behaviour on first embed.

**With more time:** Offer a pluggable embedding backend (OpenAI / Cohere / Bedrock) behind one interface and add evaluation harnesses (nDCG, faithfulness) on a fixed FAQ set.

---

## 6. Aggressive chat history truncation (Groq TPM) vs conversation quality

**Choice:** Cap history by turns and character budget before each LLM call to stay under free-tier TPM limits.

**Trade-off:** The model may “forget” earlier constraints or user preferences within a long session.

**With more time:** Summarize older turns into a rolling memory block, or persist structured booking state server-side so the model is not the system of record.

---

## 7. “Hidden” admin + optional bearer token vs full IAM

**Choice:** `/admin` is not linked in the main nav; `ADMIN_TOKEN` optionally protects `/ai/admin/*`.

**Trade-off:** Security is opt-in and UI-side hiding is not authorization.

**With more time:** OAuth2 / OIDC, role-based access, audit logs for ingest and booking changes, and rate limits on admin and chat endpoints.

---

# Lessons learned & what I would build next

**What worked well**

- Treating **tool results as the source of truth** for bookings and tightening the assistant’s wording when the model drifts.
- **Splitting services** (UI, booking API, AI) so each repo has a clear contract and can be replaced independently.
- **RAG ingestion as a real pipeline** (replace-by-source, chunk counts in the response) so operators see cause and effect.

**What was painful**

- Provider-specific quirks (token limits, malformed tool calls) required **defensive parsing** and retries—not something the happy-path tutorials emphasize.
- **Local embeddings + Qdrant** are powerful but slow down “first run” until Docker, models, and seeds are all aligned.

**Next increments (highest leverage)**

1. **Observability:** trace IDs across `hospital-ui` → `chatbot-ai` → `hospital-api`, with redacted prompts and tool payloads in structured logs.
2. **Evaluation:** golden questions for RAG (expected citations) plus a small set of adversarial booking prompts.
3. **Production auth:** real admin sessions, CSRF-safe uploads, and secrets outside `.env` in developer machines only.
4. **Deployment:** one `docker compose` stack with health checks and documented resource limits for Ollama and Qdrant.

---

See also: runtime and ingestion diagrams in [`architecture.md`](architecture.md).
