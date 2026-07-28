---
name: sarvam-mcp
description: >-
  Use the Sarvam MCP server (sarvam-mcp) from any agent harness to run Indic
  speech and language tasks live — STT, TTS, translate, transliterate, LLM,
  vision, dubbing, localization — or to fetch Sarvam API docs and snippets
  while writing code. Use when Sarvam MCP is connected, when the user asks to
  translate/transcribe/speak/dub/localize Indian-language content in-chat, or
  when setting up MCP for Cursor, Claude Code, Claude Desktop, Windsurf, Zed,
  Codex, Gemini CLI, VS Code, Cline, Continue, or LM Studio.
license: Apache-2.0
compatibility: Requires the sarvam-mcp MCP server connected in the agent client (or install via uvx/pip). Network access to api.sarvam.ai.
metadata:
  author: sarvam-ai
  version: "1.0"
---

# Sarvam MCP

Harness-agnostic guide for the official [sarvam-mcp](https://github.com/sarvamai/sarvam-mcp) server. Prefer these MCP tools over inventing HTTP/SDK calls when the user wants Sarvam to **do something now**.

> [!IMPORTANT]
> Two namespaces — pick the right one before every call:
>
> | Namespace | When | Examples |
> |-----------|------|----------|
> | `sarvam_tools_*` | **Runtime** — call Sarvam APIs live | "Translate this to Tamil", "Transcribe this audio", "Say this in Hindi" |
> | `sarvam_code_*` | **Build-time** — help write code / look up APIs | "How do I call TTS from Python?", "Which STT languages?" |
>
> Decision: **use Sarvam now** → `sarvam_tools_*`. **Write code that uses Sarvam** → `sarvam_code_*`.

For SDK-only coding without MCP, use the sibling skills: [translate](../translate), [speech-to-text](../speech-to-text), [text-to-speech](../text-to-speech), [chat](../chat), [voice-agents](../voice-agents).

## Prerequisites

1. MCP server connected as `sarvam` (or equivalent) — see [references/install.md](references/install.md).
2. API key: `SARVAM_API_KEY` in client config, or `~/.sarvam/credentials` (`api_key = sk_...`).
3. Key from [dashboard.sarvam.ai/key-management](https://dashboard.sarvam.ai/key-management).

Auth header on the wire is `api-subscription-key` — never invent `Authorization: Bearer`.

### Auth errors

If any tool returns auth failure:

1. Call `sarvam_tools_set_api_key` with no args → get dashboard link.
2. Call again with the pasted `sk_...` key → saved to `~/.sarvam/credentials`.

## Intent → tool (runtime)

| User intent | Tool |
|-------------|------|
| Transcribe audio | `sarvam_tools_stt_transcribe` |
| Speech → English text | `sarvam_tools_stt_translate` |
| Long audio / diarization | `sarvam_tools_stt_batch_submit` → `sarvam_tools_stt_batch_status` |
| Speak text (TTS) | `sarvam_tools_tts_speak` (or `_tts_stream`) |
| Translate text | `sarvam_tools_translate` |
| Script conversion | `sarvam_tools_transliterate` |
| Detect language | `sarvam_tools_identify_language` |
| Chat / complete | `sarvam_tools_llm_complete` |
| Document → structured text | `sarvam_tools_vision_extract` (+ `_vision_job_status`) |
| Audio in → spoken reply | `sarvam_tools_voice` |
| Dub audio to another Indic lang | `sarvam_tools_dub` |
| Localize i18n JSON/YAML/etc. | `sarvam_tools_localize` |
| Summarize / Q&A over audio | `sarvam_tools_recall` |
| Pronunciation dictionary CRUD | `sarvam_tools_pronunciation_*` |

Full parameter notes: [references/tools.md](references/tools.md).

## Intent → tool (build-time)

| User intent | Tool |
|-------------|------|
| Search docs | `sarvam_code_search_docs` |
| Endpoint request/response shape | `sarvam_code_api_reference` |
| Language coverage by API | `sarvam_code_languages` |
| TTS speakers for a model | `sarvam_code_speakers` |
| Copy-paste snippet | `sarvam_code_snippet` |
| Recommend model + lang | `sarvam_code_recommend_model` |
| Validate draft request body | `sarvam_code_validate_request` |
| Pricing structure | `sarvam_code_pricing` |

When writing an integration: `recommend_model` → `api_reference` / `snippet` → `validate_request` before the user ships.

## Workflows (prefer composites)

Do **not** hand-chain STT → translate → TTS when a composite exists:

| Workflow | Pipeline | Tool |
|----------|----------|------|
| Voice reply | STT → LLM → TTS | `sarvam_tools_voice` |
| Dubbing | STT → Translate → TTS | `sarvam_tools_dub` |
| i18n file | Translate string table | `sarvam_tools_localize` |
| Recall | STT → LLM summary/Q&A | `sarvam_tools_recall` |

## Agent checklist

```
- [ ] MCP connected? If not → install (references/install.md) or fall back to SDK skills
- [ ] Runtime vs build-time namespace chosen
- [ ] Auth OK (or sarvam_tools_set_api_key)
- [ ] Prefer composite workflow tools over manual chains
- [ ] Absolute paths for local audio/files
- [ ] Language codes BCP-47 (`hi-IN`, `od-IN` not `or-IN`)
- [ ] Surface result paths / transcripts; mention observability if useful
```

## Gotchas

| Gotcha | Detail |
|--------|--------|
| **Wrong namespace** | Runtime asks must not use `sarvam_code_*`. Code-writing must not burn credits on `sarvam_tools_*` unless the user wants a live demo. |
| **Tool name prefix** | Live tools are `sarvam_tools_*` / `sarvam_code_*`. Older docs may omit `_tools` / `_code` — always match the connected server's tool list. |
| **REST STT ~30s** | Longer audio → batch tools (`stt_batch_*`) or ask the user to split. |
| **TTS langs ⊂ STT langs** | STT ~23 langs; TTS ~11. Dub/voice reply language must be TTS-supported. Use `sarvam_code_languages` when unsure. |
| **Odia code** | `od-IN` — never `or-IN`. |
| **Audio paths** | Prefer absolute local paths. Many tools also accept `audio_base64` / `audio_url` + `filename`. |
| **Output location** | Generated audio/docs land under `SARVAM_MCP_BASE_PATH` (default `~/Desktop`) unless the tool returns another path. |
| **Bulbul v3 speakers** | v2 names (`anushka`, …) can 400 on v3. Default to `priya` / `shubh` or call `sarvam_code_speakers`. |
| **Translate models** | `mayura:v1` — modes + `output_script`, fewer langs. `sarvam-translate:v1` — more langs, formal-only. |
| **Don't re-implement** | If MCP is connected, call the tool — don't paste `curl` / SDK for the same action unless the user asked for code. |

## MCP unavailable

1. Install/connect per [references/install.md](references/install.md).
2. Or use SDK skills + [docs.sarvam.ai/llms.txt](https://docs.sarvam.ai/llms.txt) to write code the user can run.

## Full docs

- [sarvam-mcp README](https://github.com/sarvamai/sarvam-mcp)
- [Installation by client](references/install.md)
- [Tool reference](references/tools.md)
- [docs.sarvam.ai/llms.txt](https://docs.sarvam.ai/llms.txt)
