<p align="center">
  <img src="images/CAF-AotW-banner.svg" width="100%" alt="CAF AotW banner">
</p>

# 04/07/2026 &mdash; AotW#6: AISAC &mdash; AI Scientific Assistant Core

---

## Science Story

Scientific researchers working on HPC environments face a persistent bottleneck: translating
domain expertise into computational workflows requires navigating heterogeneous systems,
specialized tools, and large bodies of literature simultaneously — often across separate
sessions and tools. AISAC (AI Scientific Assistant Core), developed at Argonne National
Laboratory, addresses this by providing a unified natural-language interface through which
researchers can search and synthesize scientific literature (arXiv, PubMed), execute and debug
analysis code, query domain-specific knowledge bases, and orchestrate multi-step workflows —
all within a single session. AISAC is domain-agnostic and has been deployed on ALCF Aurora
and Improv for workflows spanning materials science, climate modeling, and computational
biology.

<p align="center">
  <img src="images/06-AISAC/aisac_architecture.png" width="80%" alt="AISAC Architecture">
</p>

---

## Agentic Motivation

Traditional scientific computing requires researchers to manually coordinate across
disconnected tools: literature search engines, notebooks, HPC job schedulers, and knowledge
bases. Each handoff is a friction point, and multi-step analyses — retrieve relevant papers,
extract methods, adapt code, run on HPC, interpret results — are error-prone when performed
sequentially by hand.

AISAC's agentic design directly addresses this:

- **Autonomous planning:** A Planner agent decomposes complex research queries into ordered
  steps and delegates to specialized helper agents, enabling multi-step workflows that would
  otherwise require expert manual orchestration.
- **Adaptive routing:** A Router agent classifies each query and selects the appropriate
  execution path (direct response, retrieval, planning, or tool use), avoiding unnecessary
  computation while ensuring complex tasks receive full multi-agent treatment.
- **Tool integration at scale:** Over 50 tools span web search, arXiv/PubMed literature
  retrieval, hybrid RAG over institutional documents, safe code execution (stateless and
  stateful sessions), and structured data analysis — all callable by agents without human
  intervention.
- **Cross-agent collaboration:** Via the Model Context Protocol (MCP), AISAC agents can
  discover and invoke external specialized agents in the Academy exchange, supporting
  federated scientific workflows across institutional boundaries.
- **Long-horizon memory:** Persistent SQLite + FAISS memory scoped per user and agent enables
  coherent multi-session research workflows, accumulating learned user preferences and prior
  findings across conversations.

---

## Implementation

AISAC is implemented in Python 3.10+ (~35,000 LOC) as a layered multi-agent system with a
strict driver–helper separation:

**Agent architecture (LangGraph state machine):**

```
Depth 0: Router      → classifies intent: direct | plan | retrieve | tool
Depth 1: Planner     → decomposes query into ordered steps (no tool access)
Depth 1: Coordinator → executes plan steps, dispatches to helpers
Depth 2: Helpers     → RAG Retrieval | Code Execution | Web/Literature Search | Data Analysis
```

Drivers (Router, Planner, Coordinator) operate on the control plane with no direct tool
access, enforcing clean separation between reasoning and execution. Safety limits bound
execution depth and context usage.

**Key technologies:**
- **Orchestration:** LangGraph ≥0.2 + LangChain ≥0.2
- **UI:** Gradio with live trace streaming and a real-time Inspector pane showing per-agent
  reasoning
- **Retrieval:** Hybrid BM25 (Tantivy) + semantic (FAISS) RAG with score normalization,
  per-file chunk capping, and optional MMR reranking
- **LLM providers:** Argo Legacy/Public (ANL), ALCF Sophia, OpenAI, AMSC (via LiteLLM),
  local models — unified via a single interface with endpoint probing and failover
- **Auth:** Globus PKCE flow for ALCF; no stored secrets
- **Cross-agent:** MCP bridge to Academy exchange for federated agent discovery and invocation
- **Persistence:** SQLite WAL (conversations, blackboard, preferences), FAISS
  (embedding-model-scoped indices), Tantivy indices

**Deployment:** AISAC runs on researcher laptops via a desktop launcher (tkinter + pywebview),
on HPC login nodes via terminal + SSH tunnel, and in air-gapped environments via offline mode.

<p align="center">
  <img src="images/06-AISAC/aisac_depth_model.png" width="80%" alt="AISAC Depth Model">
</p>

---

## To Know More

### Source Code
- **Repository:** Available internally via Argonne National Laboratory GitLab
- **License:** Copyright &copy; Argonne National Laboratory, UChicago Argonne LLC. All rights reserved.

### Additional Resources
- **Paper:** Bhattacharya, C., & Som, S. (2025). *AISAC: An Integrated Multi-Agent System
  for Transparent, Retrieval-Grounded Scientific Assistance.* [arXiv:2511.14043](https://arxiv.org/abs/2511.14043)
- **Contact:** Chandrachur Bhattacharya &mdash; cbhattacharya@anl.gov

---

*Last Updated: April 2026*  
*Contributed by: Chandrachur Bhattacharya, Argonne National Laboratory*
