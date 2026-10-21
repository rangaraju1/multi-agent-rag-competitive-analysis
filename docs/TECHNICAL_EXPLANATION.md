# Technical Explanation

## 1. Business problem

Manual competitor research requires framing questions, finding evidence, comparing offerings, and producing a readable report. This prototype automates those steps while keeping them separated into specialized agent responsibilities.

## 2. Design approach

The project uses a **sequential multi-agent workflow** rather than an unconstrained agent conversation. LangGraph provides explicit stage order, while `CompetitiveAnalysisState` carries intermediate results and error information between nodes.

### Question generation

The first agent invokes `suggest_questions` to identify the company's sector and create a bounded list of analysis questions. Structured output helps downstream nodes consume the questions consistently.

### Web research and storage

The second agent invokes `fetch_search_results` with the questions and uses Tavily Search to collect answer text. The `store_in_chromadb` tool skips empty answers, prepares document metadata and IDs, and stores embeddings in ChromaDB. The stored document count can be lower than the number of questions if research answers are empty or a call fails.

### Retrieval and drafting

The final node queries the Chroma vector store for company-related context, builds a report prompt using `generate_report`, and invokes the LLM. The prompt requests an executive summary, company and industry overview, competitor comparison, SWOT, recommendations, and conclusion, while discouraging unsupported numeric claims.

## 3. Model and data layer

- **LLM:** `gpt-4o-mini`, temperature 0, through an OpenAI-compatible endpoint.
- **Embeddings:** `text-embedding-3-small`.
- **Storage:** ChromaDB, with vector retrieval used in report drafting.
- **External source:** Tavily Search, accessed at runtime.

The original notebook uses a **course-provided endpoint**; access to that service is not guaranteed outside the learning environment. Porting to a personal API provider requires compatible models and authorized credentials.

## 4. Testing and troubleshooting

The notebook was tested in Google Colab with a clean runtime. During development, two important issues were resolved:

1. **Execution order:** An early workflow invocation ran before the `StateGraph` had been compiled, producing a `NameError`. Reordering/removing duplicate execution cells corrected this.
2. **Embedding authentication:** A ChromaDB insertion test initially returned HTTP 401. A direct embedding request succeeded after verifying the environment configuration; rerunning storage then succeeded. The original cause of the transient failure was not definitively established.

These tests validate the notebook prototype, not every future API response or generated factual claim.

## 5. Limitations

- Search results may be incomplete, stale, or inaccurate.
- Generated conclusions can contain errors despite retrieval grounding.
- A course-specific endpoint limits immediate reproducibility.
- This is a notebook-based proof-of-concept, not a production service.
- Source URLs, citation verification, and systematic evaluation would improve auditability.

## 6. Next steps

1. Preserve source URLs and retrieval timestamps alongside answers.
2. Add structured report evaluation (factuality, citation coverage, relevance).
3. Add API retry/backoff, caching, and failure-aware branching.
4. Package agents into testable Python modules with unit tests.
5. Expose the workflow via a Streamlit interface or API after security review.
