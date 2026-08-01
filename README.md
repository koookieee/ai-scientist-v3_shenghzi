# Supplementary Code

Three components. The agent writes papers; the two APIs are the services it calls.

```
ai_scientist/   Autonomous research agent. Runs in a container, conducts experiments,
                writes a LaTeX paper, submits it for review, and revises on the feedback.
search_api/     Literature search over ~928K arXiv CS/stat papers (vector + BM25 hybrid).
review_api/     Peer review service. Wraps a paper as a sandboxed task and returns a
                structured review with numeric scores.
```

**Flow.** `ai_scientist` drafts a paper, calls `review_api` for a review, reads the
feedback, runs further experiments, and resubmits. Each cycle is snapshotted as a
version. Both the agent and the reviewer call `search_api` for literature grounding.

## Configuration

No credentials are included. Each component reads its configuration from the
environment; see the per-component README. The endpoints default to `localhost`,
so all three can run on one host.

| Variable | Used by | Purpose |
|---|---|---|
| `ANTHROPIC_API_KEY` | ai_scientist, review_api | Agent model access |
| `GEMINI_API_KEY` | search_api | Query embeddings |
| `GEMINI_INDEX_HF_REPO` | search_api | Dataset repo holding the prebuilt index |
| `REVIEW_API_URL` | ai_scientist | Review service endpoint |
| `SEARCH_PUBLIC_URL` | ai_scientist, review_api | Search service endpoint |
| `E2B_API_KEY` | review_api | Sandbox provider (if not using Docker) |
| `GITLAB_KEY` | ai_scientist | Optional: per-run branch history |

## Prompts

All agent and reviewer instructions are plain markdown, in two places:

- `ai_scientist/.claude/` — researcher instructions (`CLAUDE.md`), review/code/idea
  subagent definitions (`agents/`), and the paper-search skill (`skills/`)
- `ai_scientist/harbor-task/instruction.md.template` — the per-run task prompt
- `review_api/prompts/` — reviewer instructions and judge prompts

`ai_scientist/ideas/` holds the 20 task specifications. Each contains the target
paper's public OpenReview reviews, used as the seed signal for revision.
