# AI Scientist

Runs an autonomous research agent inside a container. The agent reviews literature,
runs experiments, writes a LaTeX paper, submits it for external review, and revises
against the feedback. Each submission is snapshotted under `submissions/v{N}_{ts}/`.

## Run

```bash
uv sync
export ANTHROPIC_API_KEY=...  REVIEW_API_URL=...  SEARCH_PUBLIC_URL=...
./run.sh ideas/idea_06_lifelongagentbench_with_reviews.json \
    --model <model> --timeout 21600 --use-upstream-agent --env docker --gpus 1
```

`--gpus 0` for CPU-only tasks. Outputs land in `jobs/<idea>__<timestamp>/`, with the
agent trajectory and synced artifacts under `harbor-task-*/agent/`.

## Layout

```
run.sh                      Orchestrator: builds the task, launches the container, syncs artifacts
harbor-task/                Task definition
  instruction.md.template   Per-run prompt; {{IDEA_CONTENT}} and {{GITLAB_BRANCHES}} are substituted
  environment/              CPU and GPU container images
.claude/
  CLAUDE.md                 Researcher instructions (workspace, rigor requirements, review loop)
  agents/                   reviewer, code-reviewer, idea-reviewer subagents
  skills/search-papers/     Paper search CLI + docs, exposed to the agent as /app/search
ideas/                      20 task specifications
scripts/
  submit_for_review.sh      Submits to the review API, writes a versioned snapshot
  compile_latex.sh          pdflatex + bibtex + chktex
  gitlab_setup.py           Optional: one branch per run, so later runs can read earlier ones
  sanitize_secrets.py       Redacts credentials from trajectories before they are written
paper_template/             ICLR conference (full-paper) template, copied into the workspace at start
local_harbor_agents/        Agent-runner patches, incl. GPU passthrough for the container backend
```

## Notes

- `REVIEWER_MODE=api-external` routes review to the external service. The `subagent`
  and `ensemble` modes exist in the code but were not used for the reported results.
- GPU passthrough requires the patch in `local_harbor_agents/patch_docker_gpu.py`;
  `run.sh` applies it at startup.
