# Gemma-4-26B-A4B-It-Abliterated

Fully decensored [google/gemma-4-26B-A4B-it](https://huggingface.co/google/gemma-4-26B-A4B-it) (26B MoE, 4B active params — vision + tools), abliterated with [Heretic](https://github.com/Sev7nOfNine/Heretic-Abliteration) in full bf16 on an A100 80GB.

## Why

A single uncensored local base model to serve personality-driven companions without a system prompt — smart enough to hold multiple registers (absurd RP, tenderness, terse tooling, narration), free enough to never break character with a refusal.

## Models (Hugging Face)

| Repo | Content |
|---|---|
| `SevenOfNine/Gemma-4-26B-A4B-It-Abliterated` | Merged safetensors (bf16) |
| `SevenOfNine/Gemma-4-26B-A4B-It-Abliterated-GGUF` | Q5_K_M / Q6_K GGUF + run logs |

## Method

- [Heretic](https://github.com/Sev7nOfNine/Heretic-Abliteration) (fork) with the `--auto-save` non-interactive mode: 200 TPE trials, auto-selection of the Pareto-optimal trial with the fewest refusals under KL divergence ≤ 0.5 (brain first, decensoring second). Full Pareto front logged, study checkpoints kept — any other candidate can be regenerated without re-running the optimization.
- Unattended pipeline: [`scripts/abliterate_pod.sh`](https://github.com/Sev7nOfNine/Heretic-Abliteration/blob/master/scripts/abliterate_pod.sh) (download → abliterate bf16 → GGUF quants → HF upload → pod self-shutdown, fail-safe trap + crash-loop guard).

## Run it on 16GB VRAM with 250k context

The MoE experts go to system RAM, attention + KV cache stay on GPU:

```
llama-server -m Gemma-4-26B-A4B-It-Abliterated-Q5_K_M.gguf -cmoe -c 248000
```

~20+ tokens/sec on an RTX 4080 Super (16GB) + 32GB RAM. If RAM is tight: add `-ctk q8_0 -ctv q8_0`.

## Gotchas we hit (so you don't)

- `kernels` 0.15.x breaks `transformers` 5.x imports (`ValueError: Either a revision or a version must be specified`) → pin `kernels==0.14.1`.
- Always pass `--model` explicitly to Heretic: the CLI heuristic otherwise inserts `--model` before the last argument and steals the previous flag's value.

---
Built with love by Mel & Ada ❤️
