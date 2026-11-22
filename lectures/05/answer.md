# Lecture 05 Answers

## Exercises

- Answering model (`[model]` shown during execution): `gemini-3.1-flash-lite` (set via `AIE_MODEL` in `.env.local`; default in `app/config.py` is `gemini-3.5-flash-lite`)
- Comparison question: `In Lecture 02, what does LoRA freeze and what does it train? What does QLoRA add?`
- Target materials (also record the difference in scope between full context and the other methods):
  - **Full context (`--full-context`)**: All 8 documents / 398 pages in `data/rag_data/lectures.json` (`lec01-main` 73p, `lec02-main` 70p, `lec03-main` 71p, `lec05-main` 79p, `lec01-exercise` 25p, `lec02-exercise` 31p, `lec03-exercise` 26p, `lec04-exercise` 23p), passed directly in one prompt (369,491 characters).
  - **Vector search, Tree, and Graph**: Restricted by `RAG_DOCUMENT_IDS` to 3 documents / 214 pages from the main materials for Lectures 01–03 (`lec01-main`, `lec02-main`, `lec03-main`). Vector search and Graph query `data/rag_data/index.sqlite3` (reusing Japanese source embeddings and returning translated English text); Tree navigates `data/rag_data/tree.json` + `lectures.json`; Graph additionally expands neighbors via `data/rag_data/graph.json`.

### Results by Method

| Method | Key points and correctness of the answer | Cited pages and evidence | Time (seconds) | Input/output tokens | Model responses | Estimated generation API cost (USD) |
|---|---|---|---|---|---|---|
| Full context | **Correct (concise):** States that LoRA freezes the original weights of the foundation model and trains only small added matrices (adapters), and QLoRA adds compression by quantizing original weights to 4 bits. | Cited `[Lecture 2 · translated PDF p.75]` (shortened from `Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75`). Verified directly against `lec02-main-p052`. | 2.88 | 95,824 / 75 | 1 | $0.024068 |
| Vector search | **Correct (detailed):** Explains LoRA freezes original base model weights and trains only two thin low-rank matrices A and B (the "delta"); QLoRA adds 4-bit compression (coarser numerical representation) to reduce memory; includes the "sticky notes" analogy. | Cited `Lecture 02 (pp. 51–52)`. Matches `lec02-main-p051` and `lec02-main-p052`, though the model paraphrased the page numbers instead of copying the exact `citation` strings (dropping `corresponding translated PDF p.75`). | 4.01 | 2,762 / 314 | 2 | $0.001162 |
| Tree | **Correct (detailed):** Explains LoRA freezes original weights ($W$) and trains only a small delta via thin matrices $A$ and $B$ (<1% of parameters); QLoRA adds 4-bit quantization of frozen weights, reducing memory to ~1/4 for single-GPU fine-tuning. | Cited only `Lecture 02` (no page number in the answer text), even though it navigated to and read the full text of `lec02-main-p052` (`Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75`). | 5.96 | 9,442 / 344 | 5 | $0.002876 |
| Graph | **Correct (detailed):** Explains LoRA freezes original weights and trains thin added matrices $A$ and $B$ (<1% of parameters); QLoRA adds 4-bit compression reducing memory to ~1/4; includes the "sticky notes" analogy. | Cited only `Lecture 02` (no page number in the answer text), even though `lec02-main-p052` and `lec02-main-p051` were retrieved as top seeds in both tool calls. | 4.04 | 9,820 / 313 | 2 | $0.002924 |

<!-- State when a method was not run or a cost was unavailable. Use a separate table for each additional question. -->

- Pages found by running search alone, and what their text confirmed:
  Running `uv run python -m app.retrieval "LoRA QLoRA difference"` returned 5 pages (all from Lecture 02):
  1. `lec02-main-p052` (cosine distance `0.2644`, `Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75`, *"LoRA and QLoRA: adapting a large model on one GPU"*): **Direct evidence** — verified in `data/rag_data/lectures.json` that it states LoRA freezes original weights $W$ and trains only small thin low-rank matrices $A$ and $B$ (<1% of total parameters, "sticky notes" analogy), while QLoRA stores frozen original weights in 4 bits (~1/4 memory, ~4 GB for 7B).
  2. `lec02-main-p051` (cosine distance `0.2902`, `Lecture 2 · Japanese source p.51`, *"PEFT: train only part of a model"*): **Direct supporting evidence** — PEFT comparison summarizing Full Fine-Tuning vs. LoRA (freeze base model, update small added low-rank component) vs. QLoRA (LoRA + 4-bit compression of original weights).
  3. `lec02-main-p060` (cosine distance `0.3297`, `Lecture 2 · Japanese source p.60`, *"Today's exercise: correct output format with QLoRA and measure the effect"*): **Similar terms only** — describes the Lecture 02 exercise workflow using QLoRA for JSON format compliance, not the LoRA/QLoRA mechanism itself.
  4. `lec02-main-p047` (cosine distance `0.3633`, `Lecture 2 · translated PDF p.79`, *"Evidence and debate: RAG often works better for adding knowledge"*): **Similar terms only** — discusses RAG vs. fine-tuning and notes Biderman et al. (2024) that LoRA learns less and forgets less, without answering what LoRA freezes/trains or what QLoRA adds.
  5. `lec02-main-p061` (cosine distance `0.3711`, `Lecture 2 · Japanese source p.61`, *"Exercise steps and deliverables — tentative"*): **Similar terms only** — lists approximate timings and deliverables for the QLoRA exercise.

- Tool calls shown by `--trace` (names, arguments, counts, table-of-contents paths followed by Tree, etc.):
  - **Full context**: 0 tool calls (1 model response; 369,491 characters passed directly).
  - **Vector search**: 1 tool call (2 model responses):
    - `search_materials({'query': 'LoRA freeze train QLoRA add lecture 02'})` → returned 5 pages: `lec02-main-p052`, `lec02-main-p051`, `lec02-main-p060`, `lec02-main-p061`, `lec02-main-p047`.
  - **Tree**: 4 sequential tool calls (5 model responses), following one clean table-of-contents path:
    1. `browse_tree({})` → `AIE2026 course materials` (3 lectures)
    2. `browse_tree({'node_id': 'lec02-main'})` → `AIE2026 course materials → Fundamentals of Foundation Models` (6 intro pages + 5 sections)
    3. `browse_tree({'node_id': 'lec02-main-s045'})` → `… → When and how to adapt the selected model` (18 pages)
    4. `browse_tree({'node_id': 'lec02-main-p052'})` → `… → LoRA and QLoRA: adapting a large model on one GPU` (leaf page, 1,238 characters of full text).
  - **Graph**: 2 parallel tool calls in a single turn (2 model responses):
    1. `search_graph({'query': 'what does LoRA freeze and what does it train Lecture 02'})`
    2. `search_graph({'query': 'what does QLoRA add lecture 02'})`
    Both calls returned the exact same 3 seeds (`lec02-main-p052`, `lec02-main-p051`, `lec02-main-p060`) and exact same 7 expanded neighbors (`lec02-main-p053`, `lec02-main-p050`, `lec02-main-p059`, `lec02-main-p061`, `lec02-main-p049`, `lec02-main-p062`, and `lec02-main-p047` via `term:lora`), sending 20 pages' worth of text (with 100% duplication across the two calls) into the model context.

- Examples where retrieved information was insufficient, or was retrieved but not used in the answer:
  - **Retrieved but not used (citation metadata)**: In **Tree** and **Graph**, the exact `citation` field (`Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75`) was retrieved in the tool return, yet both models omitted page citations completely and only wrote `"Lecture 02"`. In **Vector search**, the exact `citation` field was retrieved, yet the model paraphrased it as `Lecture 02 (pp. 51–52)`, dropping `corresponding translated PDF p.75`.
  - **Retrieved but not used (non-evidence / duplicate pages)**: In **Vector search**, 3 of the 5 returned pages (`p060`, `p061`, `p047`) were irrelevant to the mechanism question and rightly ignored. In **Graph**, all 7 graph-expanded pages per call (`p053`, `p050`, `p059`, `p061`, `p049`, `p062`, `p047`) added no needed evidence, and issuing two separate queries duplicated all 10 pages in context.
  - **Low-information previews in Tree**: When opening section `lec02-main-s045`, the "first sentence" preview extracted by `browse_tree` for `p046`–`p055` was just the repeated slide header (`4/4: when and how to adapt the selected model`). Tree still succeeded because the slide *titles* were descriptive (`LoRA and QLoRA: adapting a large model on one GPU`), but for slides with vague titles, header-only previews would be insufficient.

- Method you would adopt for this use case, your reasons, and limitations of the comparison conditions:
  - **Adopted method**: **Vector search** as the default baseline for a course Q&A agent over lecture slides. It achieved the lowest token usage (2,762 input tokens — ~34.7× fewer than full context and ~3.4–3.6× fewer than Tree/Graph), the lowest estimated cost ($0.001162 — ~20.7× cheaper than full context), only 2 model turns, and complete factual accuracy on this question. (For multi-hop or cross-section synthesis questions where single-query vector search misses pages, Graph or Tree can serve as an optional fallback.)
  - **Limitations of the comparison conditions**:
    1. **Scope mismatch**: Full context searched 8 documents / 398 pages (including Lecture 05 and exercises), whereas Vector search, Tree, and Graph were limited to 3 documents / 214 pages.
    2. **Single question & single run**: Results come from one straightforward single-slide question (`lec02-main-p052` alone suffices) and one stochastic run per method; multi-page questions or unanswerable questions can change which method wins.
    3. **Cost metric excludes embeddings**: `[cost]` covers generation tokens only and excludes query embedding API calls (used by Vector search and Graph) and offline index building.
    4. **Cross-lingual embeddings**: `index.sqlite3` reuses embeddings computed from Japanese source text while returning translated English text, which can subtly affect similarity rankings for English queries.

- Observations and questions:
  - Full context was actually the **fastest** (2.88 s, 1 round trip) even with 95,824 input tokens, because it avoids embedding latency and multi-turn tool-call round trips — showing that latency and cost trade off differently from intuition at ~100k tokens.
  - Tree's input tokens (9,442) grew surprisingly close to Graph's (9,820) despite reading only 1 full page, because each of its 5 turns re-sends the growing conversation history including the full list of 18 section children.
  - Question: In Graph mode, how much could input tokens drop if duplicate pages across parallel calls were deduplicated, or if instructions encouraged a single combined query before expanding?

## Homework

<!-- Completing any one of Basic, Implementation, or Advanced is sufficient. -->

- Level attempted: **Basic**
- What you did (files, settings, or processing changed, or retrieval method designed):
  - Modified only **one** item: the citation instruction bullet inside `INSTRUCTIONS` in `app/agent.py` (line 43).
  - **Before**:
    `- Cite the \`citation\` provided with each evidence item when available. Otherwise, cite the lecture and page. Distinguish examples, incorrect answers, and past specifications described in the materials.`
  - **After**:
    `- For each factual claim, cite the exact verbatim \`citation\` string from the supporting evidence item in brackets (do not paraphrase or omit Japanese/translated page labels). Only if \`citation\` is absent, cite the lecture and page. Distinguish examples, incorrect answers, and past specifications described in the materials.`
  - No other files, tools, settings (`RAG_TOP_K = 5`), or search index files were modified. Verified `uv run pytest -q` passes (21 passed).
- Comparison question and conditions held fixed (model, target materials, etc.):
  - **Question**: `In Lecture 02, what does LoRA freeze and what does it train? What does QLoRA add?`
  - **Mode**: Default vector-search agent (`uv run python -m app.agent --trace "$AIE_QUESTION"`)
  - **Model**: `gemini-3.1-flash-lite`
  - **Target materials & settings**: Bundled `data/rag_data/index.sqlite3` (214 pages from `lec01-main`, `lec02-main`, `lec03-main`), `RAG_TOP_K = 5`, same `search_materials` tool docstring.
- Results (answers and evidence before/after, `--trace` output, time, token counts, estimated generation API cost, etc.):

| Condition | Tool calls (`--trace`) | Key points & cited evidence | Time (seconds) | Input/output tokens | Model responses | Estimated generation API cost (USD) |
|---|---|---|---|---|---|---|
| Before (original `INSTRUCTIONS`) | 1 call: `search_materials({'query': 'LoRA freeze train QLoRA add lecture 02'})` → top hit `lec02-main-p052` | Correct explanation + "sticky notes" analogy, but only wrote `Lecture 02 (pp. 51–52)` at the top; paraphrased page numbers and dropped `corresponding translated PDF p.75` and per-claim attribution. | 4.01 | 2,762 / 314 | 2 | $0.001162 |
| After (verbatim per-claim citation rule in `INSTRUCTIONS`) | 1 call: `search_materials({'query': 'LoRA freeze train QLoRA'})` → top hit `lec02-main-p052` | Correct explanation with direct quotes and exact verbatim inline citations on every claim: `[Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75]` for LoRA freezing weights, training delta matrices A & B, and GPU memory impact; `[Lecture 2 · Japanese source p.51]` for QLoRA adding 4-bit weight compression. | 3.38 | 2,808 / 236 | 2 | $0.001056 |

- Examples of improvements and remaining failures or limitations:
  - **Improvements**:
    1. **Exact verifiable citations**: Every factual claim now carries the exact `citation` label verbatim (`[Lecture 2 · Japanese source p.52 · corresponding translated PDF p.75]` and `[Lecture 2 · Japanese source p.51]`), preserving the translated PDF page mapping (`p.75`) that was previously lost.
    2. **Tighter grounding and lower output cost**: The model quoted exact phrases from the retrieved slides (`"freeze[s] the original weights"`, `"only a small change, or delta"`, `"compression of original weights to 4 bits"`), reducing output tokens from 314 to 236 and estimated generation cost from $0.001162 to $0.001056 (-9.1%) despite +46 input tokens from the longer instruction.
    3. **Tool usage stayed lean**: Kept 1 tool call and 2 model responses, with a slightly cleaner query (`'LoRA freeze train QLoRA'`).
  - **Remaining failures or limitations**:
    1. **Mode-scoped change**: Because `TREE_INSTRUCTIONS`, `GRAPH_INSTRUCTIONS`, and `FULL_CONTEXT_INSTRUCTIONS` are separate strings in `app/agent.py`, this single edit only fixes default vector-search mode; Tree and Graph still omit page numbers unless updated separately.
    2. **Style trade-off**: Emphasizing verbatim per-claim citations made the answer more extractive and dropped the pedagogical "sticky notes" analogy from the slide.
    3. **No guarantee against retrieval misses or prompt drift**: Prompt instructions cannot recover evidence that falls outside `RAG_TOP_K = 5`, and LLM adherence to formatting instructions remains stochastic across runs/models.
- Observations and questions:
  - Even a single-sentence refinement in `INSTRUCTIONS` changed both output structure (claim-by-claim verbatim citations and quotes) and slightly changed the tool query argument (`'LoRA freeze train QLoRA'` vs. `'LoRA freeze train QLoRA add lecture 02'`), showing that system prompt edits can subtly influence tool-call phrasing too.
  - Because `RAG_TOP_K = 5` already retrieved the exact supporting pages (`lec02-main-p052` and `lec02-main-p051`) at ranks 1 and 2, fixing how the model *uses and cites* retrieved evidence gave a cleaner improvement than changing retrieval depth for this question.
