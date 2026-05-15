# albert-spores

Private checkpoint pool for the albert. SPORE federated learning module.

## Structure
```
spores/
  {collaborator}/
    {YYYY-MM-DD}/
      spore_{epoch}_{loss}.safetensors
      spore_{epoch}_{loss}.json        ← metadata
```

## How to contribute a spore

From your training machine (after running `setup_collaborator.sh`):

```bash
python3 ~/projects/ternary-intelligence-stack/albert-moe-13/scripts/produce_spore.py \
  --name yourname \
  --spores-repo ~/spores
cd ~/spores && git push
```

The spore will appear in `spores/{yourname}/{date}/`.
The main training loop ingests it automatically on the next EvolutionManager check.

## What's in a spore

| File | Contents |
|------|----------|
| `spore_ep{N}_{loss}.safetensors` | Full model checkpoint at that epoch |
| `spore_ep{N}_{loss}.json` | Metadata: epoch, loss, corpus_mix, base_checkpoint, hardware |

The SPORE module in EvolutionManager runs a fitness gate before ingesting —
spores with loss above the current main-run loss are silently rejected.
