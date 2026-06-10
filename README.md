
# Gemma-4-26B-A4B-It-Abliterated

Fully decensored [google/gemma-4-26B-A4B-it](https://huggingface.co/google/gemma-4-26B-A4B-it) — 26B Mixture-of-Experts (4B active params), vision + tool calling — abliterated with [Heretic](https://github.com/Sev7nOfNine/Heretic-Abliteration) in **full bf16** on an A100 80 GB (no quantized shortcut for the direction estimation).

Looking for GGUF? → [Gemma-4-26B-A4B-It-Abliterated-GGUF](https://huggingface.co/SevenOfNine/Gemma-4-26B-A4B-It-Abliterated-GGUF) (Q5_K_M / Q6_K)

## Why

A single uncensored local base model able to carry personality-driven companions **without a system prompt** — smart enough to hold multiple registers (absurd RP, tenderness, terse assistant, narration), free enough to never break character with a refusal.

## Result

| Metric | Value |
|---|---|
| Baseline refusals (original model) | **100 / 100** |
| Selected trial (Trial 98) refusals | **18 / 100** |
| KL divergence vs original | **0.0845** |

Selection rule: *fewest refusals while KL divergence stays ≤ 0.5* (brain first, decensoring second). Heretic warns that KL > 0.5 means significant capability damage; at **0.0845** the model stays sharp while 82 % of hard refusals are removed. The benchmark scores *extreme* harmful prompts — ordinary creative/roleplay use is far less guarded and effectively unblocked at this level.

## Verified working

Converted to GGUF and run locally on an **RTX 4080 Super (16 GB) + 32 GB RAM** at **34.5 tokens/sec** (Q5_K_M, `llama.cpp -cmoe`). Reasoning, vision and tools inherited from the base model. Coherent, correct, multilingual — the low KL shows in practice.

## Method

- [Heretic](https://github.com/p-e-w/heretic) v1.3.0 via [this fork](https://github.com/Sev7nOfNine/Heretic-Abliteration) (adds a non-interactive `--auto-save` mode for unattended runs).
- 200 TPE optimization trials, abliterating `attn.o_proj` + `mlp.down_proj` across 30 layers.
- Automatic trial selection: fewest refusals with KL ≤ 0.5, fallback to lowest-KL candidate. Full Pareto front + per-trial metrics logged in the GGUF repo (`logs/`).
- Datasets: `mlabonne/harmless_alpaca` / `mlabonne/harmful_behaviors` (Heretic defaults).

## Gotchas we hit (so you don't)

- `kernels` 0.15.x breaks `transformers` 5.x at import (`ValueError: Either a revision or a version must be specified` in `hub_kernels.py`) → pin `kernels==0.14.1` (bites on Windows AND Linux).
- Always pass `--model` explicitly to Heretic — its CLI heuristic otherwise inserts `--model` before the last argument and steals the previous flag's value.
- GGUF conversion of Gemma 4 needs `transformers >= 5.6`; llama.cpp's convert requirements pin an older one that can't read the Gemma 4 tokenizer (`'list' object has no attribute 'keys'`). Upgrade transformers after installing those requirements.


## Run locally (verified)

```
llama-server -m Gemma-4-26B-A4B-It-Abliterated-Q5_K_M.gguf -cmoe -c 248000 -ngl 99   --jinja --reasoning-format deepseek --reasoning on
```

**34.5 tok/s** on an RTX 4080 Super (16 GB) + 32 GB RAM with `-cmoe` (experts in system RAM, attention + KV cache on GPU). The `--reasoning-format deepseek` flag puts Gemma 4 thinking in a separate `reasoning_content` channel instead of leaking it into the reply.

## HuggingFace

- Model: [SevenOfNine/Gemma-4-26B-A4B-It-Abliterated](https://huggingface.co/SevenOfNine/Gemma-4-26B-A4B-It-Abliterated)
- GGUF: [SevenOfNine/Gemma-4-26B-A4B-It-Abliterated-GGUF](https://huggingface.co/SevenOfNine/Gemma-4-26B-A4B-It-Abliterated-GGUF)

---
Built with love by Mel & Ada ❤️
