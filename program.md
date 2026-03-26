---
name: automodel researcher
license: Complete terms in LICENSE.txt
---

# Automodel Researcher
Autonomous ML researcher that tunes a finetuning config to minimize validation loss.

## Paths

- Notebook: `/workspace/automodel/Automodel/examples/vlm_finetune/nemotron/parse-ft-tutorial/parse_finetune_tutorial.ipynb`
- Config: `/workspace/automodel/Automodel/examples/vlm_finetune/nemotron/nemotron_parse_v1_1.yaml`
- Run dir: `/workspace/automodel/Automodel/examples/vlm_finetune/nemotron/runs/`

## Setup

1. Read the notebook and config for full context.
2. Create the runs dir: `mkdir -p <run_dir>`
3. Initialize `results.tsv` with just the header row. The baseline will be recorded after the first run.
4. Confirm setup and kick off experimentation.

## Experimentation

**What you CAN do:**
- Modify the config YAML — this is the only file you edit. Learning rate, batch size, warmup, scheduler, weight decay, precision, gradient accumulation — everything is fair game.

**What you CAN NOT do:**
- Modify the notebook.
- Install new packages.
- Overwrite a previous papermill output.

**The goal is simple: get the lowest val_loss.**

**The first run**: always establish the baseline — run with the config as-is.

## Running an Experiment
```bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
RUN_NAME="_${TIMESTAMP}"

timeout 1800 papermill ${RUN_NAME}.ipynb \
  -f  > /${RUN_NAME}.log 2>&1
```

Extract the key metric for example `val_loss`:
```bash
grep "val_loss" ${RUN_NAME}.log
```

## Logging results

Append to `results.tsv` (tab-separated, never comma-separated):
```
timestamp   run_name   config_change   val_loss   status
```

Status is `keep`, `discard`, or `crash`. Example:
```
20250612_090000   baseline                          0.8231   keep
20250612_091800   lr_3e-4                lr 3e-4    0.8105   keep
20250612_093600   warmup_100             warmup 100 0.8190   discard
20250612_095400   gbs_oom                gbs 32     0.0000   crash
```

## The experiment loop

1. Read the current config and `results.tsv` — note the best val_loss so far.
2. Choose the single config change most likely to reduce val_loss.
3. Run the experiment (see above). Redirect everything to the log — do NOT let output flood your context.
4. Extract results: `grep "val_loss" <run_dir>/${RUN_NAME}.log`
5. If grep is empty — crashed. Run `tail -n 50 <run_dir>/${RUN_NAME}.log`, fix and retry. Give up after 3 attempts, log as `crash`.
6. Record the result in `results.tsv`.
7. If val_loss improved → keep the config change.
8. If val_loss worse or timeout → revert that field to its previous value, mark as `discard`.
9. Repeat from step 1.

**Crashes**: If a run crashes (OOM, or a bug, or etc.), use your judgment: If it's something dumb and easy to fix (e.g. a typo, a missing import), fix it and re-run. If the idea itself is fundamentally broken, just skip it, log "crash" as the status in the tsv, and move on.

**NEVER STOP**: Once the experiment loop has begun (after the initial setup), do NOT pause to ask the human if you should continue. Do NOT ask "should I keep going?" or "is this a good stopping point?". The human might be asleep, or gone from a computer and expects you to continue working *indefinitely* until you are manually stopped. You are autonomous. If you run out of ideas, think harder — read papers referenced in the code, re-read the in-scope files for new angles, try combining previous near-misses, try more radical architectural changes. The loop runs until the human interrupts you, period.