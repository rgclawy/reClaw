# reClaw

[![Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Your open-spec alternative to OpenClaw that YOU control, not the other way around.

---

## One Command to Start

### Codex CLI
```bash
codex "$(cat prompts/kickstarter.md)"
```

### Kimi CLI
```bash
kimi --prompt "$(cat prompts/kickstarter.md)"
```

### Claude Code
```bash
claude --prompt "$(cat prompts/kickstarter.md)"
```

*(Note: Verify commands work in your environment. Commands may vary by version.)*

---

## The Problem

With agentic AI coding, codebases are exploding. OpenClaw has grown to **~500K+ lines**. 

**The risk:** You run code you don't understand. Vulnerabilities hide in thousands of files. Dependencies break without warning. You're trusting compiled binaries.

**The solution:** Open the specification. Humans should review, audit, and control what runs.

---

## Who's in Control?

| Who | Controls What |
|-----|---------------|
| **You** | What features to include or exclude |
| **Human Reviewers** | Audit before merging into main |
| **AI** | Only implements what you approve |

**You are the pilot. AI is just a faster way to type.**

---

## How reClaw Keeps You in Control

### 1. Open Spec (This Repo)
Everything is documented in human-readable markdown. Read it, question it, challenge it.

### 2. Choose Your Features
Core is minimal. Extensions are optional. You decide what's in your build.

### 3. Extensions Are Pluggable
Add or remove features anytime. No lock-in. Full control.

### 4. Your Implementation
The code is yours. You own it. You modify it. You're not dependent on our binary.

---

## The Philosophy

> **Open spec > Open code**

When the specification is open:
- You can verify behavior before running
- You understand what the system does
- Different implementations can coexist  
- No lock-in to any codebase
- Humans stay in control

---

## Language, OS & Platform Agnostic

Your choice. Not ours.

- **Language:** Python, TypeScript, Go, Rust, or any other
- **OS:** macOS, Linux, Windows, or any

The spec describes **what** to build, not **how**.

---

## What's Included

### Core Architecture ✅ (Minimal)
- Gateway Server (WebSocket, auth, sessions)
- TUI (Terminal UI, streaming chat)
- Sub-agent system (ACP)
- Memory (short-term + long-term)
- Config (YAML + $VAR)

### Extensions (Your Choice) ⚡

Choose only what you need:

| Category | Options |
|----------|---------|
| **Channels** | Discord, Telegram, WhatsApp, Slack, Signal, iMessage... |
| **Voice TTS** | OpenAI, Edge, ElevenLabs |
| **Voice STT** | Whisper, Deepgram |
| **Phone** | Twilio, Telnyx, Plivo |
| **Features** | Multi-account, DM policies, Hybrid search |

**You pick what you need. Nothing more.**

---

## You're in Control

- Read `impl/` - Understand the spec
- Check `extensions/` - Choose features  
- Review before implementing
- Skip anything you don't want

**You decide. AI assists. You control.**

---

## Skills

| Skill | Purpose |
|-------|---------|
| `reclaw-implementation` | Build reClaw from scratch |
| `reclaw-update` | Update existing implementation with latest specs |
| `reclaw-extensions` | Add/remove extensions anytime |

---

## Citation

If you use reClaw in your research, please cite:

```bibtex
@misc{reclaw,
  author       = {reClaw Contributors},
  title        = {reClaw: An open-spec alternative to OpenClaw},
  year         = {2026},
  howpublished = {\url{https://github.com/rgclawy/reclaw}},
  note         = {GitHub repository}
}
```

---

## Acknowledgments

- **Functionally inspired by** [OpenClaw](https://github.com/openclaw/openclaw) - The original open-source agent framework
- **Philosophically inspired by** [OpenAI Symphony](https://github.com/openai/symphony) - Spec-first, language-agnostic agent orchestration