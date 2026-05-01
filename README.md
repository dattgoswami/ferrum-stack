# Ferrum Platform
A composable autonomous software engineering platform where coding agents act, remember, coordinate, and measure improvement across sessions.
---

## Projects

| Layer | Project | Lang | What it does |
|---|---|---|---|
| 4 | [forge-agent](https://github.com/dattgoswami/forge-agent) | Python | Continual-learning coding agent — executes tasks via TDD, learns from outcomes, improves across runs. |
| 4 | [ferrum-agent](https://github.com/dattgoswami/ferrum-agent) | Python | Multi-agent orchestration runtime — planning, execution, review, memory, and human approval in one control plane. |
| 4 | [cl-agent](https://github.com/dattgoswami/cl-agent) | Python | Reusable CL substrate — captures episodes, replays experience, and distills reusable skills without fine-tuning. |
| 3 | [ferrum-mcp](https://github.com/dattgoswami/ferrum-mcp) | Rust | Production MCP server — typed tool surface for coding, crypto, DeFi, wallet, and semantic memory search. |
| 3 | [taste-memory](https://github.com/dattgoswami/taste-memory) | Python | Preference and persona memory — user profiles, versioned prompt assets, episodic interaction memory for agents. |
| 2 | [ferrum-memory](https://github.com/dattgoswami/ferrum-memory) | Python | Memory layer — working memory, hybrid retrieval, and prioritized experience replay buffer. |
| 2 | [ferrum-evals](https://github.com/dattgoswami/ferrum-evals) | Python | Evaluation harness — correctness, safety, trajectory quality, BWT/FWT continual-learning metrics. |
| 2 | [PersonalKB](https://github.com/dattgoswami/PersonalKB) | Python | Offline-first CLI RAG — grounded hybrid retrieval over personal technical libraries, zero API cost. |
| 1 | [axon](https://github.com/dattgoswami/axon) | Rust | Local inference server — dynamic batching, SSE streaming, three-tier fallback, TurboQuant KV cache compression. |
| 1 | [ferrum-relay](https://github.com/dattgoswami/ferrum-relay) | Rust | Async job relay — HTTP clients offload fetch/compute jobs, receive job IDs, poll for results. |
| 1 | [tokengate](https://github.com/dattgoswami/tokengate) | Rust | Token metering layer — usage tracking, exact-decimal billing, chargeback analytics for LLM API calls. |
| 0 | [ferrum-core](https://github.com/dattgoswami/ferrum-core) | Rust | Shared Rust primitives — error types, JSON logging, OpenTelemetry span export (crates.io v1.0.0). |
| 0 | [forge-core](https://github.com/dattgoswami/forge-core) | Python | Framework-agnostic domain layer — tools, schemas, guardrails, Docker sandbox, eval harness wiring. |
| 0 | [nova-api-openstack](https://github.com/dattgoswami/nova-api-openstack) | Python | OpenStack VM API proof of concept — state-machine lifecycle management, layered API design reference. |

---

## Critical Path — Minimum Viable Self-Improving Agent

```
                      TASKS.md  /  operator  /  external trigger
                                        ||
                                        ||  work requests, goals
                                        ||
 ______________________________________________________________________________
||                                                                            ||
||  LAYER 4 -- THE AGENT                                                     ||
||                                                                            ||
||  forge-agent  (Python -- pure SDK, no LangGraph, no DSL)                 ||
||                                                                            ||
||  Wake cycle:                                                               ||
||    read TASKS.md --> build WorkspaceContext --> write failing test (TDD)  ||
||    --> implement (up to 3 attempts) --> run tests --> commit if green     ||
||    --> log episode { task_id, tool_calls, test_result, reward }           ||
||        to ferrum-memory POST /experience                                  ||
||                                                                            ||
||  Dream cycle:                                                              ||
||    ferrum-memory GET /replay?strategy=prioritized&k=20                    ||
||    --> Anthropic API: reflect on failure patterns                         ||
||    --> ferrum-memory POST /memory/store  (learnings as semantic mem)      ||
||    --> write DREAMS.md  (human-readable, read at next Wake)               ||
||    --> ferrum-memory POST /session/close (consolidate working memory)     ||
||                                                                            ||
||  LLM routing:                                                              ||
||    FILE_OPS  --> axon  (local Rust/Candle inference, 80% of all calls)   ||
||    CODE_GEN  --> Anthropic API  (claude-sonnet-4-6, 20% of calls)        ||
||____________________________________________________________________________||
          ||                         ||                         ||
          || MCP tool calls          || episode writes          || reward signal
          ||                         || replay reads            ||
          ||                         ||                         ||
  ________||_________    ____________||____________    _________||_________
 ||                 ||  ||                        ||  ||                  ||
 ||  LAYER 3        ||  ||  LAYER 1               ||  ||  LAYER 2         ||
 ||                 ||  ||                        ||  ||                  ||
 ||  ferrum-mcp     ||  ||  ferrum-memory         ||  ||  ferrum-evals    ||
 ||  Rust MCP       ||  ||  Python/FastAPI         ||  ||  Python/pytest   ||
 ||  server         ||  ||  + Qdrant :6333        ||  ||  + SQLite        ||
 ||                 ||  ||  + SQLite              ||  ||                  ||
 ||  coding/* (WIP) ||  ||  + Redis  :6379        ||  ||  Operational:    ||
 ||   read_file     ||  ||                        ||  ||  -- ToolCorrect. ||
 ||   write_file    ||  ||  POST /experience       ||  ||  -- Trajectory   ||
 ||   apply_patch   ||  ||  GET  /replay          ||  ||     Score        ||
 ||   grep          ||  ||  POST /memory/*        ||  ||  -- Guardrail    ||
 ||   glob          ||  ||  POST /session/*       ||  ||     Check        ||
 ||   list_dir      ||  ||                        ||  ||                  ||
 ||   write_test    ||  ||  <-- THE replay buffer ||  ||  CL Research:    ||
 ||   run_tests     ||  ||      is the research   ||  ||  -- BWT          ||
 ||   git_commit    ||  ||      primitive --       ||  ||  -- FWT          ||
 ||   git_status    ||  ||      without it, each  ||  ||  -- Plasticity   ||
 ||                 ||  ||      session starts    ||  ||     Index        ||
 ||  crypto/* (ok)  ||  ||      cold, no CL       ||  ||  -- Reward slope ||
 ||   price         ||  ||                        ||  ||                  ||
 ||   position      ||  ||  data stores:          ||  ||  Without this,   ||
 ||                 ||  ||  Qdrant  (vectors)      ||  ||  "self-improving"||
 ||  defi/*   (ok)  ||  ||  SQLite  (episodes)    ||  ||  is a claim,     ||
 ||   yield         ||  ||  Redis   (sessions)    ||  ||  not a           ||
 ||   risk_score    ||  ||                        ||  ||  measurement     ||
 ||_________________||  ||________________________||  ||__________________||
          ||
          || shared primitives, tracing, OTel spans
          ||
 __________||___________________________________________________________________
||                                                                            ||
||  LAYER 0 -- Infrastructure (built, published to crates.io v1.0.0)        ||
||                                                                            ||
||  axon          Rust    local inference (HuggingFace Candle)               ||
||                        axon-native /v1/generate API  :3000               ||
||                        80% of forge-agent LLM calls routed here           ||
||                        three-tier fallback: GPU --> CPU --> stub          ||
||                        TurboQuant KV cache compression 7x (ICLR 2026)    ||
||                                                                            ||
||  ferrum-core   Rust    unified FaError/FaResult, fixed-schema JSON log    ||
||                        OTLP span export via #[instrument_fa]              ||
||                        no HTTP framework dep -- pure shared library       ||
||____________________________________________________________________________||
```

---

## Full Platform Architecture

```
 ______________________________________________________________________________
||                           FERRUM  PLATFORM                                ||
||  Composable autonomous software-engineering platform --                   ||
||  agents that act  *  remember  *  coordinate  *  prove improvement        ||
||____________________________________________________________________________||

               TASKS.md  /  operator  /  external workflow
                                    ||
                                    ||  work requests, approvals, goals
                                    ||
 ______________________________________________________________________________
||  LAYER 4 -- AGENT SURFACES                                                ||
||____________________________________________________________________________||
||                                                                            ||
||  forge-agent      Python                                                  ||
||    overnight TDD coding agent with continual-learning wake/dream loop     ||
||    wake:  read TASKS.md --> context --> TDD loop --> commit --> log       ||
||    dream: replay --> reflect --> store learnings --> DREAMS.md            ||
||    routes 80% of calls to axon (local), 20% to Anthropic API             ||
||    :8004  (HTTP wrapper optional in v0)                                   ||
||                                                                            ||
||  ferrum-agent     Python                                                  ||
||    multi-agent planner and orchestration control plane                    ||
||    LangGraph + FastAPI + Postgres + Redis + optional NATS                 ||
||    plan -- delegate -- execute -- review -- approve (human gate)         ||
||    A2A protocol, CloudEvents, Streamlit operator UI                       ||
||    :8003  (Month 2 -- build after forge-agent is proven standalone)      ||
||                                                                            ||
||  cl-agent         Python                                                  ||
||    reusable continual-learning substrate, framework-agnostic              ||
||    capture -- replay -- distillation -- evaluation                        ||
||    thin adapters per agent surface (Codex, LangGraph, OpenAI SDK)        ||
||    extracts the learning loop from forge-agent as a portable library      ||
||____________________________________________________________________________||
               ||                                  ||
               ||  tool calls, episodes,            ||  persona context,
               ||  plans, eval events,              ||  prompt assets,
               ||  approvals, rewards               ||  user preferences
               ||                                  ||
 ______________________________________________________________________________
||  LAYER 3 -- TOOLING AND COORDINATION                                      ||
||____________________________________________________________________________||
||                                                                            ||
||  ferrum-mcp       Rust                                                    ||
||    typed MCP tool boundary -- the action space for all agents             ||
||    transport: stdio (primary, Claude Desktop) + SSE / HTTP :3000         ||
||                                                                            ||
||    coding domain  (WIP -- not yet documented in project README):         ||
||      read_file   write_file   apply_patch   grep   glob                  ||
||      list_dir    write_test   run_tests     git_commit   git_status       ||
||                                                                            ||
||    crypto domain (CoinGecko, Solana RPC):                                ||
||      price   position                                                     ||
||                                                                            ||
||    defi domain (DeFiLlama):                                              ||
||      yield   risk_score                                                  ||
||                                                                            ||
||    memory domain:                                                         ||
||      search  (semantic search proxy to ferrum-memory)                    ||
||                                                                            ||
||  taste-memory     Python                                                  ||
||    human preference, versioned prompt assets, episodic interaction mem   ||
||    persona injection for the ferrum-agent planner node                   ||
||    approves prompt material before agents see it                         ||
||    stores: PostgreSQL + Qdrant                                            ||
||    :8001  (Sprint 2)                                                     ||
||____________________________________________________________________________||
               ||
               ||  experience writes, replay reads, knowledge lookup, scoring
               ||
 ______________________________________________________________________________
||  LAYER 2 -- MEMORY, KNOWLEDGE, AND EVALUATION                            ||
||____________________________________________________________________________||
||                                                                            ||
||  ferrum-memory    Python                                                  ||
||    working memory + episode log + prioritized experience replay           ||
||    hybrid retrieval: BM25 sparse + dense embeddings + RRF fusion         ||
||    endpoints:                                                             ||
||      POST /experience    -- store episode from agent run                 ||
||      GET  /replay        -- surface salient past episodes                ||
||      POST /memory/*      -- store / search semantic memory               ||
||      POST /session/*     -- open / close working memory window           ||
||    stores: Qdrant (vectors)  SQLite (episodes)  Redis (session state)   ||
||    :8000                                                                  ||
||                                                                            ||
||  ferrum-evals     Python                                                  ||
||    evaluation harness -- scores every CI run and overnight session       ||
||    operational: ToolCorrectnessMetric  TrajectoryScore  GuardrailCheck   ||
||    CL research: BWT (backward transfer)  FWT (forward transfer)          ||
||                 Plasticity Index         Reward curve slope              ||
||    SQLite schema: task_performance table, queryable dataset              ||
||    :8002                                                                  ||
||                                                                            ||
||  PersonalKB       Python                                                  ||
||    offline-first CLI RAG for personal technical libraries                 ||
||    Ollama + BM42 sparse + Qdrant RRF -- zero API cost, local only        ||
||    reference implementation of the retrieval pattern in ferrum-memory    ||
||                                                                            ||
||  data stores:                                                             ||
||    Qdrant   :6333   -- vector store (memories, episodes, KB embeddings)  ||
||    SQLite           -- episode records, eval scores, task_performance    ||
||    Redis    :6379   -- ephemeral session state, agent queues / pubsub    ||
||    SQLite           -- PersonalKB local index                            ||
||____________________________________________________________________________||
               ||
               ||  model calls, async compute, usage events, traces
               ||
 ______________________________________________________________________________
||  LAYER 1 -- INFERENCE, EXECUTION, AND BILLING                            ||
||____________________________________________________________________________||
||                                                                            ||
||  axon              Rust                                                   ||
||    production async AI inference server (HuggingFace Candle)             ||
||    models: Llama, Mistral, SmolLM2                                       ||
||    dynamic request batching + SSE token streaming                        ||
||    three-tier fallback engine: GPU --> CPU --> deterministic stub        ||
||    TurboQuant KV cache compression 7x lossless at 4-bit (ICLR 2026)    ||
||    context window: ~2,000 tokens stock --> ~14,000 with TurboQuant      ||
||    axon-native /v1/generate API  :3000  (not yet OpenAI-compatible)     ||
||                                                                            ||
||  ferrum-relay      Rust                                                   ||
||    lightweight async HTTP relay for fetch and compute offload            ||
||    clients: submit job --> receive job_id --> poll for result            ||
||    handles concurrency, worker dispatch, result tracking                 ||
||    useful for Python agents, shell scripts, LLM tool wrappers           ||
||    :3000                                                                  ||
||                                                                            ||
||  tokengate         Rust                                                   ||
||    LLM token metering, cost tracking, and chargeback analytics           ||
||    sits beside the app -- app still calls Anthropic/OpenAI directly     ||
||    app reports usage to tokengate via REST                               ||
||    exact decimal arithmetic for sub-cent token prices                   ||
||____________________________________________________________________________||
               ||
               ||  shared contracts, primitives, sandbox patterns, infra PoCs
               ||
 ______________________________________________________________________________
||  LAYER 0 -- SHARED PRIMITIVES                                            ||
||____________________________________________________________________________||
||                                                                            ||
||  ferrum-core       Rust                                                   ||
||    unified FaError / FaResult across all Rust services                   ||
||    fixed-schema JSON logging: { ts, level, svc, msg }                   ||
||    OTLP span export via #[instrument_fa]  (no HTTP framework dep)        ||
||    published to crates.io v1.0.0  --  default-features = false          ||
||    OTel collector :4317  /  Jaeger UI :16686                            ||
||                                                                            ||
||  forge-core        Python                                                 ||
||    framework-agnostic domain layer below orchestration                   ||
||    shared tools, schemas, guardrails, Docker sandbox, eval wiring        ||
||    portable: LangGraph  OpenAI SDK  CrewAI  Google ADK  AWS Strands     ||
||                                                                            ||
||  nova-api-openstack  Python                                               ||
||    FastAPI + SQLite OpenStack VM lifecycle API                           ||
||    state-machine-driven transitions, structured operational logging      ||
||    infrastructure control pattern proof of concept                       ||
||____________________________________________________________________________||
```

---

## Wake / Dream Cycle

```
 forge-agent  --  one overnight session
 ______________________________________________________________________________
||                                                                            ||
||  1.  Read TASKS.md                                                         ||
||      ||                                                                    ||
||      V                                                                     ||
||  2.  Build WorkspaceContext                                                ||
||      ferrum-mcp/coding/glob        --> list all repo files                ||
||      ferrum-mcp/coding/grep        --> find relevant symbols              ||
||      ferrum-mcp/coding/read_file   --> read key source files              ||
||      axon (local model)            --> summarize context cheaply         ||
||      ||                                                                    ||
||      V                                                                     ||
||  3.  Write failing test  (TDD first)                                       ||
||      Anthropic API  claude-sonnet-4-6   --> generate test                ||
||      ferrum-mcp/coding/write_test       --> persist to disk               ||
||      ||                                                                    ||
||      V                                                                     ||
||  4.  Implement solution  (max 3 attempts)                                  ||
||      Anthropic API                     --> generate implementation        ||
||      ferrum-mcp/coding/write_file      --> write source                  ||
||      ferrum-mcp/coding/apply_patch     --> or apply a diff               ||
||      ferrum-mcp/coding/run_tests       --> evaluate, compute reward      ||
||                          reward = 1.0 if all tests pass                  ||
||      ||                                                                    ||
||      V                                                                     ||
||  5.  Commit                                                                ||
||      ferrum-mcp/coding/git_commit                                         ||
||      ||                                                                    ||
||      V                                                                     ||
||  6.  Log episode                                                           ||
||      ferrum-memory POST /experience                                        ||
||        { task_id, tool_calls, test_result, reward, timestamp }           ||
||      ferrum-evals: record ToolCorrectnessMetric for this session         ||
||      ||                                                                    ||
||      V                                                                     ||
||  7.  Repeat for next task in TASKS.md                                      ||
||                                                                            ||
||  --------  when token budget expires or all tasks done  --------         ||
||                                                                            ||
||  8.  Dream cycle  (consolidation)                                          ||
||      ferrum-memory GET /replay?strategy=prioritized&k=20                  ||
||      Anthropic API: reflect on failure patterns across episodes           ||
||      ferrum-memory POST /memory/store  (distilled learnings)              ||
||      Write DREAMS.md  (human-readable, next Wake reads this)             ||
||      ferrum-memory POST /session/close  (consolidate working memory)     ||
||      ||                                                                    ||
||      V                                                                     ||
||  9.  Research data pipeline                                                ||
||      ferrum-evals:    compute BWT / FWT over all accumulated episodes    ||
||      SQLite:          append episode row to task_performance table         ||
||      cl-experiments:  queryable dataset --> arXiv preprint target         ||
||____________________________________________________________________________||
```

---

## Port Map

```
 ______________________________________________________________________________
||  Service            Port    Protocol              Consumed by             ||
||____________________________________________________________________________||
||  axon               3000    HTTP /v1/generate      forge-agent (80% LLM)  ||
||  ferrum-memory      8000    REST/JSON + FastAPI    forge-agent             ||
||                                                    ferrum-agent            ||
||                                                    ferrum-evals            ||
||  taste-memory       8001    REST/JSON + FastAPI    ferrum-agent planner   ||
||  ferrum-evals       8002    REST/JSON + OTLP       forge-agent             ||
||                                                    ferrum-agent reviewer   ||
||  ferrum-mcp         3000    MCP stdio / SSE        forge-agent             ||
||                                                    Claude Desktop          ||
||                                                    any MCP client          ||
||  ferrum-relay       3000    REST/JSON              Python agents           ||
||                                                    shell scripts           ||
||                                                    LLM tool wrappers       ||
||  ferrum-agent       8003    REST + A2A + Events    Operator UI             ||
||                                                    external agents         ||
||  forge-agent        8004    REST (optional)        ferrum-agent executor  ||
||  tokengate          --      REST/JSON              any service using LLMs  ||
||  Qdrant             6333    HTTP/gRPC              ferrum-memory           ||
||                                                    taste-memory            ||
||                                                    PersonalKB              ||
||  Redis              6379    Redis protocol         ferrum-memory (sessions)||
||                                                    ferrum-agent (queues)   ||
||  OTel collector     4317    OTLP/gRPC              all Rust services       ||
||  Jaeger UI          16686   HTTP                   observability           ||
||____________________________________________________________________________||
```

