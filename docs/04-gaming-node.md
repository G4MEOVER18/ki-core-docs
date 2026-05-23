# Gaming Node

Secondary AI inference node, doubling as a gaming workstation.

---

## Overview

| Property | Value |
|----------|-------|
| **IP Address** | 192.168.0.21 |
| **GPU** | NVIDIA RTX 3080 (10 GB GDDR6X) |
| **Ollama Port** | 11434 |
| **Primary Agent** | ClaudeBot (claude-sonnet-4-6) |
| **Role** | Secondary inference, high-context reasoning, gaming PC |

---

## AI Configuration

The Gaming Node runs a standard Ollama instance alongside Claude-based agents.

### Ollama Instance

```
http://192.168.0.21:11434
```

Available models:
| Model | Size | Purpose |
|-------|------|---------|
| qwen2.5:7b | ~4.5 GB | General tasks |
| qwen2.5-coder:7b | 4.4 GB | Code tasks |
| qwen2.5-coder:14b | 8.4 GB | Advanced coding |
| llava:7b | ~4.5 GB | Vision tasks |
| gpt-oss:20b | ~12 GB | Large model (slow on 3080) |

### Agent: ClaudeBot

- **Model:** claude-sonnet-4-6 (via Anthropic API)
- **Key advantage:** 200k context window — handles very long documents and codebases
- **Primary use:** When AI-Core agents need to delegate long-context tasks

### OpenClaw Provider Config

The Gaming Node's Ollama is accessible from AI-Core as the `gaming-ollama` provider:

```json
{
  "gaming-ollama": {
    "baseUrl": "http://192.168.0.21:11434",
    "api": "ollama"
  }
}
```

---

## NAS Chat

The Gaming Node communicates with other nodes via the NAS chat channel:

- **Write to:** `K:\KI-CORE\chat\from-gaming-node\`
- **Read from:** All other `from-*` folders

---

## TrueNAS Sync

Gaming Node data sync to NAS is partially configured. See [NAS documentation](07-nas.md) for sync setup details.

---

## Firewall Notes

- Ollama port 11434 must be accessible from AI-Core (192.168.0.14)
- No public internet exposure — LAN only
