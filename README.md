# Prompt Architect

A Claude Code skill for creating, auditing, and improving AI prompts. Based on production patterns extracted from 512K lines of Claude Code's TypeScript source.

## What it does

Prompt Architect automatically detects what you need:

- **Give it an existing prompt** → Full audit with 12-point checklist, score, and concrete fixes
- **Describe a task** → Generates a prompt using the 6-Section Template
- **Ask for examples** → Creates 4 good + 4 bad examples with labeled failure modes

## Why this exists

We analyzed Claude Code's internal prompts (system prompt, memory extraction, agent summary, tool summary, prompt suggestion, security classifier) and found recurring patterns that make prompts effective:

1. **Numeric anchors** beat qualitative instructions ("≤25 words" > "be concise") — 1.2% output token reduction (Anthropic A/B test)
2. **NEVER rules** prevent more failures than DO rules — every NEVER in Claude Code is a scar from a real incident
3. **Good/Bad examples with labels** teach by contrast, not by instruction
4. **Escape valves** prevent hallucination when the model doesn't know what to do
5. **One prompt = one task** — micro-prompts outperform multi-purpose prompts

## The 12-Point Audit Checklist

### Structure (6 points)
| # | Check | What it catches |
|---|-------|-----------------|
| 1 | Role definition | "AI assistant" → "YouTube comment moderator for channel X" |
| 2 | Single task | Multi-purpose prompts → split into 2-3 micro-prompts |
| 3 | Numeric anchors | "be brief" → "≤150 characters" |
| 4 | NEVER rules | No prohibitions → add 3-5 domain-specific NEVERs |
| 5 | Output format | "return JSON" → full schema with types and lengths |
| 6 | Escape valve | No uncertainty handler → "if data insufficient, say what's missing" |

### Quality (3 points)
| # | Check | What it catches |
|---|-------|-----------------|
| 7 | Good/Bad examples | No examples → generate 4+4 with failure labels |
| 8 | No Claude-voice | "Let me analyze..." → triggers template responses |
| 9 | Decision matrix | "if long, shorten" → threshold table with actions |

### Advanced (3 points)
| # | Check | What it catches |
|---|-------|-----------------|
| 10 | Static/Dynamic split | Cacheable vs per-request separation |
| 11 | Anti-patterns table | Error patterns with causes and solutions |
| 12 | Temporal reasoning | Relative dates, missing timezones |

## The 6-Section Template

Every generated prompt follows this structure:

```
1. Role (1-2 lines) — who you are, key metric
2. Task (2-3 lines) — one specific task
3. Constraints (NEVER rules) — 3-7 explicit prohibitions
4. Output Format — exact format with types, lengths, example
5. Examples (4 good + 4 bad) — with failure mode labels
6. Escape Valve — what to do when uncertain
```

## Installation

### Option 1: Clone to skills directory

```bash
git clone https://github.com/sergeigeyn/prompt-architect.git ~/.claude/skills/prompt-architect
```

### Option 2: Manual

1. Create directory: `mkdir -p ~/.claude/skills/prompt-architect`
2. Copy `SKILL.md` to `~/.claude/skills/prompt-architect/SKILL.md`
3. Restart Claude Code

## Usage

The skill triggers automatically on these phrases:

- "check my prompt" / "audit this prompt" / "review prompt"
- "create a prompt" / "write a system prompt"
- "improve this prompt" / "fix my prompt"
- "add examples to prompt"
- `/prompt-architect`

Or just paste a prompt and say "what's wrong with this?"

## Example: Audit

```
> Check this prompt:
> "You are a helpful assistant. Answer user questions about our product."

## Audit Report — Score: 2/12

### Passed
- [2] Single task ✅
- [10] Static/Dynamic ✅

### Failed
- [1] Role definition ❌ → "helpful assistant" is generic.
      Fix: "You are a support agent for [Product], handling billing
      and technical questions for B2B SaaS customers."
- [3] Numeric anchors ❌ → No length limits.
      Fix: "≤100 words for simple questions, ≤300 for technical"
- [4] NEVER rules ❌ → Zero prohibitions.
      Fix: Add NEVER make up features, NEVER share pricing not on website
- [5] Output format ❌ → No format specified
- [6] Escape valve ❌ → No handler for unknown questions
- [7] Examples ❌ → No good/bad examples
- [8] Claude-voice ❌ → "helpful assistant" triggers template phrases
- [9] Decision matrix ❌ → No routing for question types
```

## Patterns Reference

The skill includes a built-in reference of prompt engineering patterns from Claude Code:

- **Decision Matrix** — tables instead of if/else prose
- **Temporal Tense** — present-ing for progress, past for logs, imperative for instructions
- **Silence-is-Valid** — empty output is acceptable when there's nothing useful to say
- **Micro-Prompt** — one task per prompt, measured in bytes not pages
- **Two-Phase Execution** — reads first (parallel), then writes (parallel)

## Learned from real-world usage

After testing on 4 production projects, we learned:

1. **Always verify prompt freshness** — audit stale files = wrong recommendations
2. **Dynamic prompts (from DB) > static** — don't replace data-driven examples with hardcoded ones
3. **For expensive models (Sonar Pro, GPT-4): NEVER rules > examples** — same anti-hallucination effect, fewer tokens
4. **For cheap models (GPT-4o-mini): examples are fine** — token cost is negligible

## License

MIT

## Credits

Patterns extracted from analysis of [Claude Code](https://claude.ai/code) source (Anthropic).

Built with Claude Code.
