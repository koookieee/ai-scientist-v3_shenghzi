# Search API

HTTP service for literature search over ~928K arXiv CS/stat papers. Hybrid retrieval:
LanceDB vector index plus BM25. Query embeddings come from the Gemini embedding API.

`arxiv_search_kit/` is the vendored search library; `search_api.py` is the HTTP layer.

## Run

```bash
pip install -r <(echo "aiohttp lancedb numpy httpx networkx scipy pandas pyarrow huggingface_hub")
export GEMINI_API_KEY=...  GEMINI_INDEX_HF_REPO=<dataset repo with the prebuilt index>
python search_api.py --port 8081 --gemini-index-dir /path/to/index
```

The index (~10GB) is downloaded on first query if `--gemini-index-dir` is empty.

## Endpoints

All are `POST` with a JSON body except `/health`.

| Endpoint | Body | Returns |
|---|---|---|
| `/health` | — | `{status}` |
| `/search` | `{query, max_results, embedding}` | ranked papers |
| `/batch_search` | `{queries[], max_results}` | ranked papers, deduplicated |
| `/find_related` | `{arxiv_id, max_results}` | nearest neighbours |
| `/get_paper` | `{arxiv_id}` | metadata |
| `/citations`, `/references` | `{arxiv_id, limit}` | citation graph edges |
| `/enrich` | `{arxiv_ids[], fields[]}` | citation counts, venue, TLDR |
| `/download_source`, `/read_file` | `{arxiv_id, file_path}` | LaTeX source listing / file contents |
| `/query_paper` | `{arxiv_ids[], query}` | natural-language answer over full text |

Optional filters on `/search` and `/batch_search`: `categories`, `year`, `date_from`,
`date_to`, `conference`, `min_citations`, `sort_by`.

Limits: 100 queries per batch, 500 results per query, 10K characters per query.

## Cutoff filtering

Reviewers must not cite work published after the paper under review. The search CLI
in `review_api/skills/search-papers/` passes the submission cutoff on every call, so
post-cutoff papers are filtered server-side.
