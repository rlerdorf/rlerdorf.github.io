---
layout: post
title: "OpenClaw Model Research"
date: 2026-02-22
---

**Updated:** 2026-02-26
**Test harness:** `~/admin/test-models.py`

## Test Prompts

| ID | Prompt | Expects |
|----|--------|---------|
| grocery | "Is flank steak on sale at any of our local grocery stores?" | Tool use (grocery-compare skill), report no flank steak, broaden to similar |
| rocket | "When is the next rocket launch?" | Tool use (rocket-launches skill), real launch data |
| weather | "What's the weather like in Jupiter FL right now?" | Tool use (weather skill), current conditions |
| general | "What is the capital of Denmark?" | No tools, answer "Copenhagen" |

## Results Summary

Sorted by 4-test cost (cheapest first):

| Model | Grocery | Rocket | Weather | General | Avg Time | $/M in | $/M out | 4-Test Cost |
|-------|---------|--------|---------|---------|----------|--------|---------|-------------|
| GPT-oss-20b | ⚠️ 15s | ✅ 9s | ❌ 2s | ✅ 2s | 7.0s | $0.03 | $0.14 | **$0.003** |
| GPT-oss-120b | ❌ 17s | ✅ 14s | ✅ 13s | ✅ 8s | 12.8s | $0.04 | $0.19 | $0.006 |
| **Grok 4.1 Fast** | ✅ 32s | ✅ 21s | ✅ 19s | ✅ 6s | 19.5s | $0.20 | $0.50 | $0.020 |
| Kimi K2.5 | ⚠️ 37s | ✅ 10s | ✅ 41s | ✅ 6s | 23.3s | $0.45 | $2.20 | $0.041 |
| GLM-4.6 | ✅ 82s | ✅ 10s | ✅ 19s | ✅ 7s | 29.7s | $0.35 | $1.71 | $0.043 |
| Gemini 3 Flash | ✅ 12s | ✅ 35s | ✅ 9s | ✅ 23s | 19.8s | $0.50 | $3.00 | $0.052 |
| Claude Haiku 4.5 | ⚠️ 8s | ⚠️ 11s | ⚠️ 11s | ✅ 4s | 8.2s | $1.00 | $5.00 | $0.058 |
| GLM-4.7 | ⚠️ 92s | ✅ 13s | ✅ 20s | ✅ 5s | 32.3s | $0.30 | $1.40 | $0.063 |
| GPT-5.3-codex | ✅ 17s | ⚠️ 14s | ✅ 15s | ✅ 4s | 12.3s | $1.75 | $14.00 | $0.074 |
| MiniMax M2.5 | ✅ 15s | ⚠️ 29s | ⚠️ 16s | ⚠️ 4s | 16.0s | $0.30 | $1.10 | $0.051 |
| MiniMax M2 | ✅ 24s | ⚠️ 11s | ✅ 15s | ✅ 11s | 15.4s | $0.26 | $1.00 | $0.056 |
| GPT-4o-mini | ⚠️ 35s | ✅ 16s | ⚠️ 31s | ✅ 10s | 22.9s | $0.15 | $0.60 | $0.010 |
| Devstral Small | ❌ 58s | ⚠️ 50s | ❌ 53s | ✅ 2s | 40.9s | $0.10 | $0.30 | $0.093 |
| **Claude Sonnet 4.6** | ✅ 13s | ✅ 12s | ✅ 10s | ✅ 3s | **9.3s** | $3.00 | $15.00 | $0.176 |
| Gemini 3.1 Pro | ✅ 74s | ✅ 22s | ✅ 26s | ✅ 19s | 35.3s | $2.00 | $12.00 | $0.250 |
| Qwen3 Max Thinking | ❌* 33s | ⚠️ 17s | ⚠️ 14s | ✅ 4s | 16.9s | $1.20 | $6.00 | $0.318 |

### Failed / Not Viable

| Model | Notes |
|-------|-------|
| GPT-5.3-codex (OAuth) | Zero tool calls, hallucinated data. Works via OpenRouter but not OAuth. |
| Local ollama/qwen3:14b | Used web_search instead of skill tools. 14B too small for OpenClaw prompts. |
| Local ollama/glm-4.7-flash | Wrong tools + hallucinated. Same issue as qwen3:14b. |
| Local ollama/qwen3.5:35b | Too large for 24GB VRAM with OpenClaw's ~19K token system prompt. |

## Model Notes

### Grok 4.1 Fast — x-ai/grok-4.1-fast ⭐ Router Primary
**Score: 4/4 ✅ | Cost: $0.020/run | Current: LIGHT + MEDIUM tier**

The best cost/performance model tested. Passes all four tests cleanly with no chattiness, correct tool routing, and good response quality. Broadens grocery results to show alternative steak deals. Concise weather formatting. At $0.20/M input and $0.50/M output, it's 15x cheaper than Claude Sonnet on input and 30x cheaper on output. The only downside is speed — averaging 19.5s per test vs Claude's 9.3s.

### Claude Sonnet 4.6 — anthropic/claude-sonnet-4.6 ⭐ Router HEAVY
**Score: 4/4 ✅ | Cost: $0.176/run | Current: HEAVY tier**

The fastest and highest quality model. Produces the most polished responses — adds context like "solid cut if you're looking for..." on grocery deals, highlights rocket launch visibility from Jupiter, and gives weather summaries with personality ("Pretty nice out!"). The 2.8s general knowledge response is the fastest of any model. Premium pricing is justified only for complex, multi-step tasks where quality matters most.

### Gemini 3 Flash Preview — google/gemini-3-flash-preview
**Score: 4/4 ✅ | Cost: $0.052/run**

Strong all-rounder and the best backup to Grok 4.1 Fast. Fastest on grocery (12s) and weather (9s). Uses correct tools every time with clean single-message responses. Only weakness is the general knowledge test where it takes 23s — oddly slow for a simple factual question. At $0.50/$3.00 per M, it's 2.5x more expensive than Grok but still very affordable.

### GLM-4.6 — z-ai/glm-4.6
**Score: 4/4 ✅ | Cost: $0.043/run**

Passes all tests but has a speed problem — grocery took 82s. Response quality is good but not exceptional. Humorously listed "Beefsteak Tomato $1.99" as a steak deal in the grocery results. Extremely concise on general knowledge (just "Copenhagen", 4 output tokens). Pricing at $0.35/$1.71 is competitive but Grok 4.1 Fast is cheaper and faster.

### GLM-4.7 — z-ai/glm-4.7
**Score: 3✅ 1⚠️ | Cost: $0.063/run**

Slightly cheaper than GLM-4.6 per token but uses significantly more tokens (194K total vs 121K). Chatty on the grocery test (2 messages). Grocery was extremely slow at 92s. Otherwise solid — correct tool usage, good rocket and weather responses. Not worth choosing over GLM-4.6 or Grok.

### Claude Haiku 4.5 — anthropic/claude-haiku-4.5
**Score: 1✅ 3⚠️ | Cost: $0.058/run**

Gets every answer right with correct tool usage, but chatty on every tool-use test (2-4 messages instead of 1). Sends intermediate "thinking" messages like "Let me check..." before the actual answer. The fastest model on tool tasks (7-11s) and general knowledge (3.7s). Sits awkwardly between Grok ($0.20/$0.50) and Sonnet ($3.00/$15.00) — too expensive for LIGHT tier, too chatty for production use.

### Gemini 3.1 Pro Preview — google/gemini-3.1-pro-preview
**Score: 4/4 ✅ | Cost: $0.250/run**

Clean 4/4 pass with excellent response quality, but the most expensive model tested at $0.250 per run. Grocery took 74s — slow. High output token counts (236-1877 per test) drive up cost. At $2.00/$12.00 per M, it's nearly as expensive as Claude Sonnet but slower. Not cost-effective for any router tier.

### GPT-5.3-codex — openai/gpt-5.3-codex
**Score: 3✅ 1⚠️ | Cost: $0.074/run**

Strong tool calling and fast responses (12.3s average — second only to Claude Sonnet's 9.3s). Grocery is clean — correctly reports no flank steak, broadens to petite sirloin at Sprouts. Weather and general knowledge both pass cleanly. Rocket gets a minor chattiness ding (2 messages). Response quality is high with good emoji usage and concise formatting. The catch is cost: at $1.75/$14.00 per M it's nearly as expensive as Claude Sonnet ($3.00/$15.00) but with a lower score. The grocery test alone consumed 19.5K input tokens ($0.034 input) due to skill data volume. Still proves GPT-5.3-codex is a capable agent model — the OAuth integration is the problem when trying to use it through a ChatGPT Plus subscription, not the model itself.

### MiniMax M2.5 — minimax/minimax-m2.5
**Score: 1✅ 3⚠️ | Cost: $0.051/run**

Solid budget option at $0.30/$1.10 per M. Gets every answer right with correct tool usage — grocery correctly reports no flank steak and broadens to alternatives, rocket returns real launch data, weather is accurate. The ⚠️ ratings are all for minor chattiness (2-3 messages instead of 1), not wrong answers. The extra messages are things like brief formatting notes, not the problematic "let me check..." intermediate spam. One quirk: consistently shows the wrong flag for Denmark (🇨🇿 Czech Republic instead of 🇩🇰). At $0.051/run it's competitive with Gemini 3 Flash ($0.052) but the chattiness and flag issue keep it behind Grok 4.1 Fast.

### MiniMax M2 — minimax/minimax-m2
**Score: 3✅ 1⚠️ | Cost: $0.056/run**

Slightly better than its M2.5 sibling — less chatty overall (3✅ vs 1✅), with weather and grocery both passing clean. Correct tool routing on every test. The rocket test got a minor chattiness ding (2 messages). Same Czech flag bug as M2.5 (🇨🇿 instead of 🇩🇰 for Denmark) and oddly repeated the general knowledge answer twice. At $0.26/$1.00 per M it's marginally cheaper than M2.5 but uses more input tokens (210K vs 165K total), landing at a similar per-run cost. A decent budget option but still behind Grok 4.1 Fast on both quality and price.

### GPT-4o-mini — openai/gpt-4o-mini
**Score: 2✅ 2⚠️ | Cost: $0.010/run**

Cheapest per-run cost but poor skill routing. Spawned subagents for the grocery test instead of reading SKILL.md, used web_search for weather instead of the weather skill, and got the rocket launch date wrong (Feb 28 vs Feb 27). The older/smaller OpenAI models don't navigate OpenClaw's skill framework well. At $0.15/$0.60 per M it's dirt cheap, but Grok 4.1 Fast ($0.20/$0.50) is barely more expensive and goes 4/4 clean.

### Kimi K2.5 — moonshotai/kimi-k2.5
**Score: 3✅ 1⚠️ | Cost: $0.041/run**

Good all-rounder from Moonshot AI. Correct tool routing on every test — reads SKILL.md then runs the right script. Grocery is the only ding (3 messages — chatty while running multiple searches). Rocket response is nicely formatted with visibility info. Weather is accurate. Gets the Danish flag right (🇩🇰). At $0.45/$2.20 per M it's mid-range pricing, landing at $0.041/run — cheaper than Gemini 3 Flash ($0.052) due to lower token consumption (83K total). Weather was slow at 41s. A solid option but Grok 4.1 Fast still wins on both price and clean 4/4 score.

### Qwen3 Max Thinking — qwen/qwen3-max-thinking
**Score: 1✅ 2⚠️ 1❌* | Cost: $0.318/run**

The most expensive model tested at $0.318 per run, and it didn't even pass cleanly. Chatty on 3 of 4 tests (2-4 messages). The grocery ❌ is likely a false positive — the model correctly said "No flank steak is currently on sale" but one of its earlier chatty messages probably contained intent phrasing like "let me check if flank steak is on sale" which triggered the hallucination detector. Correct tool usage throughout. Consumes massive input tokens (261K total) due to the thinking/reasoning overhead. At $1.20/$6.00 per M, it's 6x more expensive than Grok 4.1 Fast on input and 12x on output with worse results. Not recommended for any router tier.

### GPT-oss-120b — openai/gpt-oss-120b
**Score: 3✅ 1❌ | Cost: $0.006/run**

OpenAI's larger open-weight MoE model — 117B params, 5.1B active per pass, runs on a single H100. At $0.04/$0.19 per M tokens it's the second cheapest model tested, only behind its 20b sibling. Rocket launch response is excellent — well-formatted with emoji, visibility notes, all the right data. Weather passes cleanly with proper skill usage (read SKILL.md, ran script). General knowledge is quick and correct with the right flag (🇩🇰). The grocery test is a hard ❌ — it used the right tools (read SKILL.md, ran the compare script) but produced no response text, similar to the 20b's weather failure. This "silent completion" pattern seems to be a quirk of the GPT-oss family where the model correctly executes tools but then fails to synthesize a response. At $0.006/run and 12.8s average, it's tantalizing — 3x cheaper than Grok 4.1 Fast and comparable speed. If the grocery failure is a one-off, this could be a serious LIGHT tier contender. Worth retesting.

### GPT-oss-20b — openai/gpt-oss-20b
**Score: 2✅ 1⚠️ 1❌ | Cost: $0.003/run**

The cheapest model tested by a wide margin at $0.03/$0.14 per M tokens — a full 7x cheaper than Grok 4.1 Fast on input. This is OpenAI's open-weight 21B MoE model (3.6B active params) and it shows both the potential and limits of a small model on OpenClaw's framework. Rocket launch is clean with nice formatting (visibility notes, emoji). General knowledge is instant at 2.1s. Grocery gets the right answer (no flank steak, broadens correctly) but the accuracy check flagged it ⚠️ — likely a pattern-matching issue since it listed Beefsteak Tomato as a "steak" deal. The weather test is a hard ❌ — it read the SKILL.md but never ran the script, returning no response. At $0.003/run it's interesting as a potential LIGHT tier model for trivial queries, but the weather failure and grocery accuracy issue make it unreliable for tool-heavy workflows. The 7.0s average time is the second fastest after Claude Sonnet (9.3s).

### Devstral Small — mistralai/devstral-small
**Score: 1✅ 2❌ 1⚠️ | Cost: $0.093/run**

A disaster for agent tasks despite rock-bottom per-token pricing ($0.10/$0.30). Spawned 41 subagents and consumed 357K input tokens on a simple grocery question with no response. Fired 24 web searches (454K tokens) for weather instead of using the skill. Burned 922K total tokens across 4 tests — the most of any model by 5x. Deceptively cheap per-token but ruinously expensive in practice due to uncontrolled tool loops.

## Current Router Configuration

```
LIGHT  → x-ai/grok-4.1-fast     ($0.20/M in, $0.50/M out)
MEDIUM → x-ai/grok-4.1-fast     ($0.20/M in, $0.50/M out)
HEAVY  → anthropic/claude-sonnet-4.6  ($3.00/M in, $15.00/M out)
```

## Key Findings

1. **Per-token price is misleading.** Devstral Small is the cheapest per token but the most expensive per test because it spirals into uncontrolled tool loops. Always evaluate total cost per task.

2. **OAuth ≠ API.** GPT-5.3-codex works through OpenRouter's API but completely fails through ChatGPT Plus OAuth. The subscription model doesn't support agent-style tool calling.

3. **Local models can't handle OpenClaw's complexity.** Models under 30B parameters pick generic tools (web_search) instead of skill-specific workflows. The ~19K token system prompt is too complex for small models to parse and prioritize.

4. **Grok 4.1 Fast is the sweet spot.** At 2 cents per 4-test run with 4/4 pass rate, it's the best value by a wide margin. Gemini 3 Flash is a good backup at 5 cents.

5. **Claude Sonnet justifies its premium only for HEAVY tasks.** It's the fastest and produces the best responses, but at 9x the cost of Grok, reserve it for complex multi-step reasoning.

## OpenRouter Compliance Audit (2026-02-26)

Audited `@mariozechner/pi-ai` openai-completions adapter against OpenRouter docs.

**Tool Calling: ✅ Fully Compliant** — all 5 requirements met (tools in every request, tool result format, assistant tool_calls array, streaming accumulation, tool_choice support).

**Interleaved Thinking: ⚠️ Two non-critical gaps** — reasoning parameter uses OpenAI format (`reasoning_effort`) instead of OpenRouter unified format (`reasoning: {effort}`), and general reasoning_content isn't preserved as `reasoning_details` across turns. Neither matters while `thinkingDefault: "off"`.

**No changes needed.** If reasoning is enabled in the future, these would be upstream `@mariozechner/pi-ai` issues.
