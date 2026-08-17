# Hermes Fish Audio TTS

[![CI](https://github.com/xiaoyaner-home/hermes-fishaudio-tts/actions/workflows/ci.yml/badge.svg)](https://github.com/xiaoyaner-home/hermes-fishaudio-tts/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-purple.svg)](LICENSE)

A Hermes-native [Fish Audio](https://fish.audio/) text-to-speech provider. It uses Hermes Agent's public `TTSProvider` plugin API—no monkey patches and no Hermes core-file edits.

## Features

- Native `PluginContext.register_tts_provider()` integration for directory and pip entry-point plugins
- Fish Audio voice cloning via `reference_id`
- `s2.1-pro-free`, `s2.1-pro`, and `s2-pro`
- MP3, WAV, PCM, and Opus output
- Configurable temperature, top-p, sample rate, bitrate, latency, speed, and volume
- Buffered synthesis plus a streaming iterator for compatible future/live voice pipelines
- Zero runtime dependencies beyond Hermes and the Python standard library
- Tested against Hermes PluginManager, TTS dispatch contracts, and standalone entry-point install discovery

## Requirements

- Hermes Agent with the native TTS provider plugin API (Hermes v0.20; v0.19 remains compatible)
- Python 3.10+
- A Fish Audio API key

## Install

```bash
hermes plugins install xiaoyaner-home/hermes-fishaudio-tts
hermes plugins enable fishaudio-tts
```

Put the credential in the active Hermes profile's `.env`:

```env
FISH_AUDIO_API_KEY=your_key_here
```

Then select Fish Audio in `hermes tools`, or configure it directly:

```yaml
plugins:
  enabled:
    - fishaudio-tts

tts:
  provider: fishaudio
  fishaudio:
    model: s2.1-pro-free
    reference_id: your_fish_reference_id
    format: mp3
    sample_rate: 44100
    mp3_bitrate: 128
    temperature: 0.7
    top_p: 0.7
```

Restart the Hermes CLI/gateway or start a new session after installing or changing the provider.

## Compatibility matrix

| Plugin version | Hermes baseline | Install/discovery path | Status |
|---|---|---|---|
| `0.2.1` | Hermes v0.20.0 / `v2026.8.3` TTS API (also compatible with v0.19.x) | `hermes plugins install`, copied plugin directory, or pip entry point `hermes_agent.plugins` | Supported; native registration, dispatch, voice-compatible conversion, and real MP3 synthesis verified |
| `0.2.0` | Hermes v0.19.1 / `v2026.7.30` TTS API | `hermes plugins install`, copied plugin directory, or pip entry point `hermes_agent.plugins` | Supported maintenance baseline |
| `0.1.x` | Hermes v0.18 native TTS provider API | Copied plugin directory | Legacy; upgrade recommended for Hermes v0.19 standalone install smoke coverage |

## Configuration

All behavior settings belong under `tts.fishaudio`; only the API key belongs in `.env`.

| Key | Default | Notes |
|---|---:|---|
| `model` | `s2.1-pro-free` | Fish Audio model header |
| `reference_id` | — | Fish voice/reference ID |
| `format` | `mp3` | `mp3`, `wav`, `pcm`, or `opus` |
| `temperature` | `0.7` | Sampling temperature |
| `top_p` | `0.7` | Nucleus sampling |
| `sample_rate` | `44100` | Output sample rate |
| `mp3_bitrate` | `128` | MP3 bitrate in kbps |
| `opus_bitrate` | — | Optional Opus bitrate |
| `speed` | — | Prosody speed multiplier |
| `volume` | — | Prosody volume |
| `normalize_loudness` | — | Normalize output loudness |
| `latency` | — | Fish latency mode |
| `language` | — | Optional language hint |
| `timeout` | `120` | HTTP timeout in seconds |
| `api_base` | `https://api.fish.audio` | Custom Fish-compatible endpoint |
| `chunk_size` | `8192` | Bytes yielded by `stream()` |

## Use

The normal Hermes tool works unchanged:

```text
Generate an audio version of: 你好，我是 Hermes。
```

The provider writes the requested audio file and returns its absolute path through Hermes' existing `text_to_speech` result contract.

### Streaming caveat

The plugin implements `TTSProvider.stream()` and yields Fish Audio response bytes incrementally. Hermes' current standard `text_to_speech` path still writes a complete file before platform delivery. True chunk-to-Discord voice-channel playback requires a generic streaming consumer in Hermes core; this plugin does not patch the gateway to simulate it.

## Development

Clone Hermes and this repository, then run the tests with the Hermes checkout on `PYTHONPATH`:

```bash
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-fishaudio-tts
PYTHONPATH=/path/to/hermes-agent /path/to/hermes-agent/.venv/bin/python -m pytest --import-mode=importlib --rootdir=tests tests
```

A real API smoke test is intentionally not run in public CI because it requires a credential and incurs an external request. Maintainers run it before releases.

For local Hermes v0.20 compatibility verification, run both the plugin suite and the host TTS registration/registry/dispatch tests against the same Hermes checkout. Use an isolated temporary `HERMES_HOME` for install or synthesis smoke tests; do not change the live `tts.provider` during release validation.

## Security

- Never commit `FISH_AUDIO_API_KEY` or generated private voice samples.
- The plugin blocks tool-call extras from overriding authorization headers or the API endpoint.
- API response errors are bounded and redact the request credential and configured Fish reference/voice ID before surfacing.

Please report vulnerabilities privately through GitHub's security advisory interface.

## Related work

The community project [poogas/hermes-fish-tts-plugin](https://github.com/poogas/hermes-fish-tts-plugin) is an earlier Fish Audio integration that uses runtime monkey-patching. This project is an independent implementation built on Hermes' native TTS provider API.

## License

MIT © 2026 Xiaoyaner
