# Automodel-Researcher
An autonomous AI agent that tunes NeMo AutoModel fine-tuning configs to minimize validation loss — without human intervention.

You point it at a notebook + config, and it runs an indefinite experiment loop: establish a baseline, tweak hyperparameters, run papermill, read the results, log everything, repeat. It stops when you stop it.

The agent tracks all experiments in a structured results.tsv and never overwrites previous runs.

## References

- [NeMo AutoModel docs](https://docs.nvidia.com/nemo/automodel/latest/index.html)
- [NeMo AutoModel](https://docs.nvidia.com/nemo/automodel/latest/index.html) environment
- [AGENTS.md](https://agents.md/) — the open format for guiding coding agents
- [How to write a great agents.md](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) — lessons from 2,500+ repositories
- [Anthropic Skills](https://github.com/anthropics/skills/blob/main/skills/algorithmic-art/SKILL.md) — example skill definitions for Claude
- [papermill](https://papermill.readthedocs.io/) for notebook execution