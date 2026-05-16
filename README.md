# albert-spores

**albert.** is a self-growing language model trained collectively — across dedicated GPUs, laptops, and everything in between. This repo is where contributors submit checkpoint fragments called *spores*. Each accepted spore gets blended into the live model at the next training cycle.

No GPU required. Any Linux or macOS machine can contribute.

---

## Setup

One time, takes about 10 minutes on first run (Rust compilation).

### 1. Authenticate GitHub

```bash
gh auth login
```

Opens a browser, takes 30 seconds. Do this **before** cloning — this repo is private.

If `gh` is not installed, get it at [cli.github.com](https://cli.github.com).

### 2. Clone and install

```bash
gh repo clone eriirfos-eng/albert-spores ~/projects/albert-spores
bash ~/projects/albert-spores/install.sh
```

Installs dependencies (git, Rust), builds the training binary, and adds three commands to `~/bin`. Subsequent runs are instant.

### 3. Open a fresh terminal

`~/bin` is now on your PATH. All three commands are available everywhere.

---

## Contributing a spore

```bash
albert-train    # trains on your CPU — stop any time with Ctrl-C
albert-spore    # packages your checkpoint and pushes it to this repo
```

`albert-spore` auto-detects your GitHub login, copies the checkpoint into this repo, and pushes. Your spore lands in `spores/{you}/{date}/` and gets picked up automatically by the main training loop if it clears the fitness gate.

You can run `albert-spore` after any Ctrl-C — partial training counts.

---

## Commands

| Command | What it does |
|---|---|
| `albert-train` | CPU training with live dashboard — Ctrl-C saves your checkpoint |
| `albert-test` | Chat with albert. locally, run benchmarks |
| `albert-spore` | Package your latest checkpoint and push it to this repo |

---

## Fitness gate

Spores are accepted when `loss < main_best + 1.0`. The margin is intentionally wide — CPU-trained checkpoints around loss 11 are welcome. The model benefits from routing diversity across varied hardware even when raw loss is far from the GPU frontier.

---

## What happens at ingestion

The SporeManager blends accepted spores into the live model at epoch boundaries with α = 0.08:

- F32 weights: `w = 0.92 · w_main + 0.08 · w_spore`
- Ternary weights: blended, then re-ternarized at ±0.04

Your checkpoint shifts the balance without overriding it. The main model wins all sign-flip contests.

---

## Spore structure

```
spores/
  {contributor}/
    {YYYY-MM-DD}/
      spore_ep{N}_{loss}.safetensors    — checkpoint weights
      spore_ep{N}_{loss}.json           — metadata (epoch, loss, architecture)
```

---

## Troubleshooting

**`albert-spore` says "no training checkpoint found"**
Training saves a checkpoint after the first complete epoch. Let `albert-train` run until you see the first `=== Epoch done ===` line, then Ctrl-C if you want to stop.

**`albert-spore` says "already committed"**
Your checkpoint is already in the repo from a previous run. Train for another epoch and try again — you'll get a new spore with a lower loss.

**Push rejected**
The repo has new commits. Run:
```bash
git -C ~/projects/albert-spores pull
```
Then run `albert-spore` again.

**Dashboard shows red indicators**
Navigate to the full URL printed by `albert-train` in the terminal — it includes the parameters the dashboard needs for CPU training speed.

**Build fails / `cargo` not found**
Re-run the installer — it installs Rust automatically:
```bash
bash ~/projects/albert-spores/install.sh
```

---

## Updating

```bash
git -C ~/projects/albert-spores pull && bash ~/projects/albert-spores/install.sh
git -C ~/projects/ternary-intelligence-stack pull
```

Safe to re-run at any time. Re-running the installer after a pull picks up any changes to the training binary or commands.
