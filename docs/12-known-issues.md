# Known Issues & Open Problems

This document tracks unresolved issues, limitations, and areas under active development.

---

## 🔴 Critical

### 1. OpenClaw Gateway → RTX 3060 Routing Failure

**Status:** Under investigation  
**Affected:** Nexus agent, `ollama-3060` provider  
**Symptom:** New gateway instances (any PID after the initial working instance) never route inference requests to port 11435 (RTX 3060). All Nexus requests fall back to port 11434 (RTX 5060 Ti), defeating the dual-GPU setup.  
**Evidence:** `ollama-3060.log` shows only `/api/ps` health checks, never `/api/chat` — even though direct calls to `http://127.0.0.1:11435/api/chat` work correctly.  
**Workaround:** Use `openclaw agent --agent nexus --local` (embedded mode, bypasses gateway) or set Nexus primary to `ollama/qwen2.5-coder:7b` (routes to 5060 Ti instead).  
**Root cause hypothesis:** Gateway model deduplication — both Ollama instances expose identical model lists (`/api/tags`). The gateway may silently prefer the first registered `ollama` provider when model IDs collide.  
**Fix needed:** Investigate gateway source (`dist/index.js`) for provider model deduplication logic, or give RTX 3060 exclusive models.

---

### 2. Gateway Bootstrap Overhead (Session Preprocessing)

**Status:** Partially mitigated  
**Symptom:** Even with a fresh session (no history), the OpenClaw gateway takes 5–15 minutes to respond before sending the first inference request to Ollama.  
**Root cause:** Gateway builds a large bootstrap context from workspace files (AGENTS.md, MEMORY.md, SOUL.md, IDENTITY.md, TOOLS.md, HEARTBEAT.md, skill descriptions). This preprocessing happens synchronously before the LLM call.  
**Mitigations applied:**
- Reduced workspace docs (IDENTITY.md, TOOLS.md, HEARTBEAT.md) to minimal size
- `compaction.reserveTokensFloor = 1000` (down from 20000) to prevent early compaction triggers
- Auto-Recovery session cleanup: JSONL files >3 days deleted, sessions.json entries >24h removed
**Still needed:** Further reduce workspace doc total size; profile gateway bootstrap time to identify the bottleneck.

---

### 3. Session Compaction on Old UUIDs

**Status:** Resolved for new sessions  
**Symptom:** Sessions created with `reserveTokensFloor=20000` continue triggering compaction even after config change, because the stored session metadata retains old `contextWindow` values.  
**Fix:** Delete `sessions.json` to force new UUID generation. Auto-Recovery script now enforces 24h session cleanup.

---

## 🟡 Medium

### 4. CyberNode TrueNAS Sync Pending

**Status:** Not yet configured  
**Symptom:** CyberNode (192.168.0.63) has not been added to the TrueNAS sync pipeline.  
**Impact:** CyberNode data is not backed up to NAS; NAS chat channel not accessible from CyberNode.  
**Fix needed:** Mount `\\192.168.0.5\Ki-Daten` as K: drive on CyberNode, configure sync task.

### 5. Gateway Config Validation Rejects `timeoutSeconds` on Providers

**Status:** Won't fix (CLI version limitation)  
**Symptom:** Adding `timeoutSeconds` to individual provider configs in `openclaw.json` causes CLI validation error:  
```
models.providers.ollama-3060: Unrecognized key: "timeoutSeconds"
```
**Root cause:** Gateway binary (v2026.5.19) supports the key, but the CLI validator (v2026.4.14) rejects it.  
**Workaround:** Only configure `timeoutSeconds` at the global `agents.defaults` level.

### 6. ComfyUI Port Mapping

**Status:** Active  
**Note:** ComfyUI container maps `0.0.0.0:8188 → 18188`. The external port (8188) differs from the container's internal port (18188). Clients must connect to port 8188.

---

## 🟢 Minor / Cosmetic

### 7. Auto-Recovery Sessions.json Race Condition (Fixed)

**Status:** Fixed  
**Fix:** Sessions.json cleanup now skips execution if any `.lock` files exist in the agent sessions directory, preventing cleanup during active sessions.

### 8. Auto-Recovery Killed Busy Gateway (Fixed)

**Status:** Fixed  
**Fix:** Two-stage health check: TCP port check (is process alive?) + HTTP /healthz check (is it responsive?). Gateway is only restarted on "crashed" state (TCP gone), not "busy" (port open but HTTP slow).

### 9. Ollama `keep_alive` Default Expiry

**Status:** Known behavior  
**Note:** Nexus warmup sets `keep_alive: -1` which Ollama translates to an expiry of `2318-09-01`. This is expected — model stays in VRAM indefinitely until evicted by another model.

---

## 📋 Feature Requests / Future Work

- [ ] Fix gateway routing to RTX 3060 (critical for dual-GPU utilization)
- [ ] Profile and optimize gateway bootstrap time (<60s target)
- [ ] CyberNode TrueNAS sync setup
- [ ] Gateway debug logging (currently no gateway.log)
- [ ] Multi-node orchestration via NAS chat (Jarvis ↔ ClaudeBot ↔ Claude Code)
- [ ] OpenClaw gateway update to v2026.5.x (supports `timeoutSeconds` per provider)
- [ ] Automated model VRAM warm-up on system boot
- [ ] Unified monitoring dashboard (all nodes + services in one view)
