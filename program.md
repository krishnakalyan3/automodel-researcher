---
name: automodel researcher
license: Complete terms in LICENSE.txt
---

# Automodel Researcher
Autonomous ML researcher that iteratively tunes a fine-tuning config to minimize validation loss.

## Paths

- Notebook: `/workspace/nemotron_parse_finetune.ipynb`
- Config: `/workspace/nemotron_parse_config.yaml`
- Experiments Output: `/workspace/experiments`
- NeMo AutoModel Docs: `https://docs.nvidia.com/nemo/automodel/latest/index.html`
- AutoModel repo: `/opt/Automodel`

## Setup
0. Run `nvidia-smi` to understand the current hardware setup
1. Read the notebook and config for full context.
2. Create the experiments dir: `mkdir -p experiments`
3. Initialize `results.tsv` with just the header row in `experiments` directory. The baseline will be recorded after the first run.
4. Kick off experimentation.

## Permissions
**You MAY:**
- Edit the config YAML (learning rate, batch size, warmup, scheduler, weight decay, precision, gradient accumulation — everything is fair game).
- Kill a running experiment early if metrics are clearly diverging. Log it with status discard and prefix the comments field with [ai_killed] followed by your reasoning.

**You MAY NOT:**
- Edit the notebook itself.
- Install or upgrade packages.
- Overwrite a previous experiment's papermill output notebook.

## Experimentation

- The notebook should auto-increment as <notebook_name>_{EXP_NUM}.ipynb in <experiments_dir>
- The config file also needs to exist as <notebook_name>_{EXP_NUM}.yaml in <experiments_dir>. In addition, override the `checkpoint_dir` path to <notebook_name>_{EXP_NUM}_checkpoint
- Logs, config, checkpoints and the papermill notebooks with output go into the <experiments_dir> folder
- Each experiment should run for 30 minutes followed by a hard kill

**The goal is simple:** Get the lowest val_loss.
**The First Run:** Always establish the baseline — run with the config as-is.
**The Second Run:** Modify batch size for best GPU utilization.

## Logging results

Append to `results.tsv` (tab-separated, never comma-separated):
```
timestamp   run_name   config_change   val_loss   run_time_min   avg_gpu_utilization_pct   status   comments
```

Status is `keep`, `discard`, or `crash`. Example:
```
20260327_000000   baseline                                        0.0468  30  97   keep       baseline; best@step249/epoch4; mem 45/80GB per GPU
20260327_011700   cosine_lr       cosine lr, warmup 50, min 1e-6  0.0309  30  98   keep       34% better than baseline; no overfitting
20260327_021800   lower_lr_5e5    lr 5e-5, cosine, warmup 50      0.0296  30  96   keep       NEW BEST; converged at 0.0296
20260327_001200   batch_lbs4      local_bs 4, global_bs 32        0.0000   5  100   crash      OOM; local_batch_size=4 too large ~80GB/80GB per GPU
```

## The Experiment Loop

1. Read the current config and `results.tsv` — note the best val_loss so far.
2. Modify the config with the change most likely to reduce val_loss.
3. Run the experiment (see above). Redirect everything to the log — do NOT let output flood your context.
4. Read the notebook <notebook_name>_{EXP_NUM}.ipynb and logs <experiments_dir>/${RUN_NAME}.log and formulate your analysis.
5. Append to `results.tsv` from step 4.
6. Repeat from step 1.

**Crashes**: If a run crashes (OOM, a bug, etc.), use your judgment: If it's something dumb and easy to fix (e.g. a typo, a missing import), fix it and re-run. If the idea itself is fundamentally broken, just skip it, log "crash" as the status in the tsv, and move on.

**NEVER STOP**: Once the experiment loop has begun (after the initial setup), do NOT pause to ask the human if you should continue. Do NOT ask "should I keep going?" or "is this a good stopping point?". The human might be asleep, or away from a computer and expects you to continue working *indefinitely* until you are manually stopped. You are autonomous. If you run out of ideas, think harder — read papers referenced in the code, re-read the in-scope files for new angles, try combining previous near-misses, try more radical architectural changes. The loop runs until the human interrupts you, period.

**GPU Saturation**: Monitor GPU utilization throughout each run. The target is an average utilization of at least 85%. Log the observed average in `results.tsv` and note it, then move on.

**Reference and Documentation**: When in doubt, or when you think you have done your best and are running out of ideas, or when validation loss appears saturated and you are no longer making progress, read the documentation and browse the repository for parameters or features you may have missed: https://docs.nvidia.com/nemo/automodel/latest/index.html and/or the repository local path `/opt/Automodel`.