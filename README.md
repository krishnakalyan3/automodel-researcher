# Automodel-Researcher
An autonomous AI agent that tunes NeMo AutoModel fine-tuning configs to minimize validation loss — without human intervention.

You point it at a notebook + config, and it runs an indefinite experiment loop: establish a baseline, tweak hyperparameters, run papermill, read the results, log everything, repeat. It stops when you stop it.

The agent tracks all experiments in a structured results.tsv and never overwrites previous runs.

## Setup

Make sure the notebook, helper configuration, and configuration Python files are mounted under the `/workspace/` folder before starting.

## Repository structure

```
.
├── program.md                              # Agent instructions (the "brain")
├── nemotron_parse_finetune.ipynb           # Training notebook (read-only)
├── nemotron_parse_dataset.py               # Dataset helper
├── nemotron_parse_config.yaml              # Base config (agent edits copies)
└── experiments/                            # Created by the agent
    ├── results.tsv                         # Experiment log
    ├── nemotron_parse_finetune_1.ipynb     # Papermill output (baseline)
    ├── nemotron_parse_finetune_1.yaml      # Config snapshot
    ├── nemotron_parse_finetune_1.log       # Stdout/stderr
    ├── nemotron_parse_config_1_checkpoint/ # Checkpoint dir
    └── ...                                 # Subsequent experiments
```

## References

- [NeMo AutoModel](https://docs.nvidia.com/nemo/automodel/latest/index.html) - Documentation
- [AGENTS.md](https://agents.md/) - the open format for guiding coding agents
- [How to write a great agents.md](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) - lessons from 2,500+ repositories
- [Anthropic Skills](https://github.com/anthropics/skills/blob/main/skills/algorithmic-art/SKILL.md) - example skill definitions for Claude
- [papermill](https://papermill.readthedocs.io/) - notebook execution