# 04b-alt — Retriever Configuration: Keyword Search (Alternative Path)
 
> **Applies to:** 04b — NASK: ResolutionFinderInternalData → Step 7b (Tool Inputs)
> **Use this guide if:** Your instance does not support Semantic search in the Retriever tool configuration
 
---
 
## When to Use This Guide
 
When configuring the **RetrieveRelevantKBContent** Retriever tool in Step 7b of [04b — ResolutionFinderInternalData](04b-nask-resolution-finder-internal-data.md), the standard instructions call for **Semantic** search criteria with the ServiceNow Embedding (E5) model, semantic indexes on `body` and `title`, and chunking with re-ranking.
 
However, on some lab instances you may encounter the following when you reach the Tool Inputs screen:
 
![Retriever Tool Inputs — Probable Defect: Semantic Indexes Required but Unavailable](../screenshots/nask-retriever-ui-probabledefect.png)
 
**Symptoms that indicate you need this alternative path:**
 
- The **Search Criteria** defaults to **Hybrid** and the **Semantic Indexes** fields are highlighted in red as mandatory — but no semantic indexes are available to select
- A note appears stating _"The retriever will support search for only E5 embeddings"_ — but the Semantic Indexes picker is empty
- The **Search sources** and **Fields returned** fields are empty even after selecting the Search profile
- Selecting **Semantic** as the Search Criteria still requires Semantic Indexes that cannot be populated
 
> **This is a known configuration limitation on some lab instances** where the E5 embedding model has not been fully provisioned or the semantic indexes on the `kb_knowledge` table have not been created. Rather than blocking progress, switch to **Keyword** search — which does not require embedding models or semantic indexes — and continue with the lab.
 
---
 
## Step 7b (Alternative): Configure Tool Inputs — Keyword Search
 
Replace the Semantic search configuration from the main 04b guide with the following Keyword search configuration.
 
**Core search configuration:**
 
| Field | Value |
|-------|-------|
| Search query | `{{GenerateSearchQueryAgainstAISearch.response}}` |
| Search space type | `Search-profile-based` |
| Search profile | `Quick Action - KB Search Profile` |
| Search sources | `kb_knowledge` |
| Fields returned | `kb_knowledge.text`, `kb_knowledge.short_description`, `kb_knowledge.category`, `kb_knowledge.description`, `kb_knowledge.article_type` |
| Limit | `10` |
| Search Criteria | **`Keyword`** |
| Rewrite query | Unchecked |
 
![Retriever Tool Inputs — Keyword Search: Core Configuration (Top)](../screenshots/NASKResolutionFinderUsingInternalData-keywordsearch1.png)
 
![Retriever Tool Inputs — Keyword Search: Core Configuration (Bottom)](../screenshots/NASKResolutionFinderUsingInternalData-keywordsearch2.png)
 
> **Key difference from Semantic:** With **Keyword** search selected, the Semantic Indexes and Embedding model fields are not required and do not appear. The retriever uses traditional keyword matching (TF-IDF / BM25 style) against the `kb_knowledge` table rather than cosine similarity in embedding space. This means retrieval quality depends more heavily on the quality of the search query — which is why the upstream `CreateOptimalSearchQuery` skill is especially important in this configuration.
 
**What is NOT needed for Keyword search:**
 
| Field | Status |
|-------|--------|
| Embedding model | Not required — no embedding needed for keyword matching |
| Semantic Indexes | Not required — keyword search uses inverted indexes, not vector indexes |
| Chunking and Re-Ranking | Not available for Keyword search — the retriever returns full document matches ranked by keyword relevance |
| Document matching threshold | Not applicable — keyword search uses its own relevance scoring |
 
Click **Continue** to proceed to Step 7c (Tool Outputs).
 
---
 
## Impact on Downstream Behaviour
 
Switching from Semantic to Keyword search changes how the retriever matches KB articles, but does **not** change the rest of the skill flow. The output format (`Rag Results` as `json_object`) remains the same, and the `Assess if solution exists` prompt still receives the same data structure.
 
| Aspect | Semantic Search | Keyword Search |
|--------|----------------|----------------|
| Matching method | Cosine similarity in E5 embedding space | Keyword frequency / BM25 relevance |
| Handles synonyms | Yes — semantically similar terms match | Limited — requires exact or stemmed keyword overlap |
| Requires embedding model | Yes — E5FT | No |
| Requires semantic indexes | Yes — `body`, `title` | No |
| Chunking and re-ranking | Available | Not available |
| Query quality sensitivity | Moderate — embeddings compensate for imprecise queries | High — keyword search is more literal, so query quality matters more |
 
> **Practical impact for the lab:** With Keyword search, the `CreateOptimalSearchQuery` skill's output becomes even more critical. The LLM-generated query must contain the exact keywords present in the KB articles (e.g., "Veritas", "backup failure", "error 84") for keyword matching to succeed. If you find that the retriever returns no results during testing, review the query output from `CreateOptimalSearchQuery` and ensure it contains specific, concrete terms from the KB article rather than abstract descriptions.
 
---
 
## After This Step
 
Once you have configured the Keyword search Tool Inputs and clicked **Continue**, rejoin the main 04b guide at **Step 7c — Tool Outputs**. All subsequent steps (Tool Outputs, Tool Conditions, Summary, Final Canvas, Publish, Activate) are identical regardless of whether you used Semantic or Keyword search.
 
→ Return to [04b — Step 7c: Tool Outputs](04b-now-assist-skill-kit-part2-resolutionfinderinternaldataskill.md)