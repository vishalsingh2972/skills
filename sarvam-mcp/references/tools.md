# Sarvam MCP — tool reference

Call schemas are authoritative on the connected server. This file is a routing map + defaults, not a full OpenAPI dump. For exact shapes while coding, use `sarvam_code_api_reference`.

## Runtime — `sarvam_tools_*`

### Speech-to-text

| Tool | Role | Notes |
|------|------|-------|
| `sarvam_tools_stt_transcribe` | File → transcript | Default model `saaras:v3`. Modes: `transcribe`, `translate`, `verbatim`, `translit`, `codemix`. REST suited to short clips (~30s). |
| `sarvam_tools_stt_translate` | Speech → English text | Dedicated speech-translation path. |
| `sarvam_tools_stt_batch_submit` | Long audio / diarization job | Use for files beyond REST limits; diarization + multi-file. |
| `sarvam_tools_stt_batch_status` | Poll / fetch batch job | Pair with submit; download when complete. |

Audio input (common pattern): `audio_path` and/or `audio_base64` / `audio_url` + `filename`. Prefer absolute paths.

### Text-to-speech

| Tool | Role | Notes |
|------|------|-------|
| `sarvam_tools_tts_speak` | Text → audio file | Default `bulbul:v3`. Returns path (and/or resource) per `SARVAM_AUDIO_OUTPUT_MODE`. |
| `sarvam_tools_tts_stream` | Text → streamed audio | Lower latency path when the client supports stream handling. |

Defaults: speaker often `priya` / `shubh`. Pace works on v3; don't invent `pitch`/`loudness`. Prefer native-script Indic text.

### Text

| Tool | Role | Notes |
|------|------|-------|
| `sarvam_tools_translate` | Cross-language translate | Models: `mayura:v1` (default in many flows) or `sarvam-translate:v1`. |
| `sarvam_tools_transliterate` | Script conversion | Roman ↔ native, etc. |
| `sarvam_tools_identify_language` | LID + script | Good pre-step before TTS/translate. |
| `sarvam_tools_text_analytics` | Typed Q&A over text | Structured analytics prompts. |

### LLM / vision / pronunciation

| Tool | Role | Notes |
|------|------|-------|
| `sarvam_tools_llm_complete` | Chat completions | Default `sarvam-30b`; flagship `sarvam-105b`. OpenAI-style messages. |
| `sarvam_tools_vision_extract` | Document intelligence | Submit job / extract. |
| `sarvam_tools_vision_job_status` | Poll vision job | |
| `sarvam_tools_pronunciation_list` | List dicts | bulbul:v3 |
| `sarvam_tools_pronunciation_get` | Get dict | |
| `sarvam_tools_pronunciation_create` | Create dict | |
| `sarvam_tools_pronunciation_delete` | Delete dict | |

### Composite workflows

| Tool | Pipeline | Required inputs (typical) |
|------|----------|---------------------------|
| `sarvam_tools_voice` | STT → LLM → TTS | Audio in; optional `system_prompt`, `reply_language`, `speaker`, `llm_model` |
| `sarvam_tools_dub` | STT → Translate → TTS | Audio + `target_language_code` (TTS langs only) |
| `sarvam_tools_localize` | Translate string table | `source_path` + `target_language_code`; optional `output_path`, `model` |
| `sarvam_tools_recall` | STT → LLM Q&A/summary | `question` + audio `paths` |

### Auth

| Tool | Role |
|------|------|
| `sarvam_tools_set_api_key` | First-time setup / rotation → `~/.sarvam/credentials` |

## Build-time — `sarvam_code_*`

These do **not** call generation APIs for the user's content (except live-verified snippets where documented). Safe default when drafting integrations.

| Tool | Role |
|------|------|
| `sarvam_code_search_docs` | Search docs.sarvam.ai |
| `sarvam_code_api_reference` | Request/response for a known endpoint path |
| `sarvam_code_languages` | BCP-47 list for `stt` / `tts` / `translate` / … |
| `sarvam_code_speakers` | Speakers for `bulbul:v3` / `v2` / beta |
| `sarvam_code_snippet` | Tested snippet (`python` / `javascript` / `typescript` / `curl`) |
| `sarvam_code_recommend_model` | Task description → model + lang heuristics (no API key) |
| `sarvam_code_validate_request` | Validate draft body before send |
| `sarvam_code_pricing` | High-level billing structure (confirm on dashboard) |

## Observability

Runtime tool responses typically include an `observability` object (latency, request IDs, credit usage). Surface it when debugging failures or cost; omit from casual answers.

## Env knobs (server)

| Variable | Default | Meaning |
|----------|---------|---------|
| `SARVAM_API_KEY` | — | Required unless credentials file set |
| `SARVAM_API_BASE_URL` | `https://api.sarvam.ai` | Override for staging |
| `SARVAM_MCP_BASE_PATH` | `~/Desktop` | Where files are written |
| `SARVAM_AUDIO_OUTPUT_MODE` | `files` | `files` \| `resources` \| `both` |
