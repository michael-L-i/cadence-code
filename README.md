# Cadence Code

[![CI](https://github.com/michael-L-i/cadence-code/actions/workflows/ci.yml/badge.svg)](https://github.com/michael-L-i/cadence-code/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.11-3.14](https://img.shields.io/badge/Python-3.11--3.14-blue.svg)](https://www.python.org/)

Natural voice conversations with Codex and Claude Code, fully local on Apple
Silicon.

Talk through a bug, redirect a task, or hear a quick update when your hands are
busy or your eyes need a break. Your coding agent still does the thinking;
Cadence Code simply handles speech.

- **Local speech:** TTS and STT run on your Mac with MLX.
- **Natural turns:** speak, listen, interrupt, and continue without reloading
  models.
- **Explicit control:** Cadence Code listens only after you start a conversation.
- **No extra reasoning layer:** no daemon, cloud speech service, or second
  language model.

## Quick start

Requires an Apple Silicon Mac running macOS 14+, Python 3.11-3.14, a microphone,
and an output device. Internet access is needed for installation and the first
model downloads.

### Claude Code

```text
/plugin marketplace add michael-L-i/cadence-code
/plugin install cadence-code@cadence-code-marketplace
```

Restart Claude Code, then run:

```text
/cadence-code:start-talking
```

### Codex

```bash
codex plugin marketplace add michael-L-i/cadence-code
codex plugin add cadence-code@cadence-code-marketplace
```

Start a new Codex session, then run `$start-talking` or choose **Start Talking**
from `/skills`.

> Codex plugins work in the CLI and desktop app. The IDE extension does not yet
> support them.

On first run, Cadence Code selects recommended defaults, asks for microphone
access, and downloads the speech models automatically. If Claude Code is still
finishing the one-time setup, run `/reload-plugins` when prompted and start
again.

## Talking with Cadence Code

| Action | Claude Code | Codex |
| --- | --- | --- |
| Start a conversation | `/cadence-code:start-talking` | `$start-talking` |
| Interrupt and redirect | `/cadence-code:jump-in` | `$jump-in` |
| Change speech models | `/cadence-code:voice-settings` | `$voice-settings` |
| End and release models | `/cadence-code:wrap-up` | `$wrap-up` |

You can also say "stop" or "goodbye" to end a conversation.

## How it works

Cadence Code runs as a lightweight stdio MCP server beside your coding agent.
It loads local speech models only when a conversation starts, keeps them warm
between turns, and releases them when the conversation ends.

Raw audio and speech inference stay on your Mac. Your transcript is returned to
Codex or Claude Code, which decides what to do and composes the exact response
spoken back to you. See [Cadence Code privacy](PRIVACY.md) for the complete
boundary.

Only one Cadence Code conversation can use the microphone and model memory at a
time.

<details>
<summary><strong>Voice models and download sizes</strong></summary>

### Text to speech

| Model | Tier | Language | Download |
| --- | --- | --- | ---: |
| [Pocket TTS 100M](https://huggingface.co/mlx-community/pocket-tts) | Lightweight | English | 236 MB |
| [Kokoro 82M](https://huggingface.co/mlx-community/Kokoro-82M-bf16) | Lightweight | English | 389 MB |
| [Chatterbox Turbo 350M](https://huggingface.co/mlx-community/chatterbox-turbo-4bit) | Balanced | English | 417 MB |
| [Qwen 0.6B](https://huggingface.co/mlx-community/Qwen3-TTS-12Hz-0.6B-CustomVoice-8bit) | Highest quality | English | 1.97 GB |

### Speech to text

| Model | Tier | Language | Download |
| --- | --- | --- | ---: |
| [Moonshine Base 61M](https://huggingface.co/UsefulSensors/moonshine-base) | Lightweight | English | 248 MB |
| [Parakeet 110M](https://huggingface.co/mlx-community/parakeet-tdt_ctc-110m) | Balanced | English | 459 MB |
| [Parakeet 0.6B v3](https://huggingface.co/mlx-community/parakeet-tdt-0.6b-v3) | Highest accuracy | 25 languages | 2.51 GB |

Pocket TTS and Parakeet 110M are the defaults. TTS is currently English-only;
choose Parakeet 0.6B v3 for multilingual transcription. Follow each linked
model card for its license terms.

</details>

## Configuration

Use the voice settings command to choose models. Advanced model, voice, speed,
endpointing, and audio-device settings live in `config.toml`:

- Codex and direct development: `~/.cadence-code/config.toml`
- Claude Code: the plugin's data directory

Model weights use the shared Hugging Face cache at
`~/.cache/huggingface/hub`. Depending on your choices, downloads require about
500 MB to 4.5 GB.

<details>
<summary><strong>Troubleshooting</strong></summary>

- **No microphone or output:** allow mic access for Codex or Claude Code in
  **System Settings > Privacy & Security > Microphone**, check the configured
  audio devices, and restart the host.
- **Setup or download fails:** check your Python version, internet connection,
  and free disk space, then restart the host to retry.
- **Session already in use:** stop Cadence Code in every Codex, Claude Code, and
  development session.
- **An update still shows the old version:** fully exit every host process that
  loaded Cadence Code, then start a new one.

For checkout-based diagnostics:

```bash
uv run --locked cadence-code doctor
uv run --locked cadence-code listen-test
```

</details>

<details>
<summary><strong>Updating and uninstalling</strong></summary>

To update, rerun the marketplace and plugin update commands for your host, then
fully restart it. A running MCP process is not replaced in place.

To uninstall:

```bash
# Codex
codex plugin remove cadence-code@cadence-code-marketplace
codex plugin marketplace remove cadence-code-marketplace

# Claude Code
claude plugin uninstall cadence-code@cadence-code-marketplace
claude plugin marketplace remove cadence-code-marketplace
```

Configuration, private environments, and shared model weights may remain after
uninstalling. Codex data lives in `~/.cadence-code`; model weights live in the
Hugging Face cache.

</details>

## Development

`uv.lock` is the source of truth for local development and CI.

```bash
uv sync --locked --python 3.13
uv run --locked cadence-code doctor
./dev check    # tests + plugin validation
./dev claude   # local branch test in Claude Code
./dev codex    # local branch test in Codex
```

See [AGENTS.md](AGENTS.md) for the project map,
[CONTRIBUTING.md](CONTRIBUTING.md) before changing dependencies, and
[RELEASING.md](RELEASING.md) for release details.

## Community

Questions and ideas are welcome in
[GitHub Discussions](https://github.com/michael-L-i/cadence-code/discussions).
Use the [issue forms](https://github.com/michael-L-i/cadence-code/issues/new/choose)
for bugs and feature proposals.

Cadence Code is released under the [MIT License](LICENSE). Please follow the
[Code of Conduct](CODE_OF_CONDUCT.md), and report security issues privately as
described in [SECURITY.md](SECURITY.md).
