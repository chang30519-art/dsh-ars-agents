# dsh-ars-agents

Academic research agent presets for [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (DSH). Seven presets: one supervisor plus six specialists, selectable from the preset picker in any new session.

This is not "an agent that reads a skill pack". It is a **team that delegates to itself**: the supervisor walks a 10-stage pipeline and hands each stage to the research / screener / writer / reviewer / verify / experiment specialist.

[中文说明](README.zh.md)

---

## The seven presets

| # | Preset | Role | Responsibility |
|---|---|---|---|
| 1 | **ARS·Main** | 10-stage orchestrator | Picks the right workflow, dispatches per stage, runs integrity gates at 2.5 and 4.5 |
| 2 | **ARS·Research** | Deep research | 6 phases, 13 roles, 8 modes; three mandatory Devil's Advocate checkpoints |
| 3 | **ARS·Screener** | Literature screening | Title/abstract → full text → conflict adjudication, auditable exclusions |
| 4 | **ARS·Writer** | Manuscript drafting | IMRAD, every citation verified, `[CITATION-NEEDED]` markers instead of guesses |
| 5 | **ARS·Reviewer** | Reviewer simulation | Editorial verdict ACCEPT / MINOR / MAJOR / REJECT with line-level comments |
| 6 | **ARS·Verify** | Integrity audit | Citation existence, claim-to-citation alignment, claim-strength drift, token conservation |
| 7 | **ARS·Experiment** | Experiment execution | Design, batch runs, result attribution, figures |

The supervisor ships a `dispatch` tool restricted by `allowedAgentPresets` to exactly the six specialists above, so it cannot wander off to unrelated agents.

## Requirements

### 1. DSH 0.2.0-rc.2 or newer

Earlier versions do not support the current `agent-preset` schema.

### 2. `academic-research-suite` skill pack (**required, not bundled**)

The personas reference that suite's workflow structure and role-document paths (for example `<skills-root>/ars/deep-research/WORKFLOW.md`).

Upstream:

```
https://github.com/Imbad0202/academic-research-skills
```

By Cheng-I Wu, licensed **CC-BY-NC-4.0** (attribution + non-commercial).

Install it so that this path exists:

```
<DSH_HOME>/skills/academic-research-suite
```

If it lives elsewhere, set `DSH_ARSS_SKILLS_DIR` to the **parent directory that contains all skills** (not the skill itself).

> This package does **not** bundle that suite: its licence requires each user to honour the non-commercial terms themselves, and it is 46 MB across 2756 files — a poor fit for a plugin package.

### 3. Reader local library (**optional**)

If you use [Reader](https://github.com/cynicalight/reader) to manage papers, these agents treat it as the first evidence layer: `reader search` the local library first, only going to the web when the library has nothing.

- The Reader app must be running (`reader status` reports connection state)
- `reader` on PATH works directly; otherwise set `READER_CLI` to `reader.exe`
- Without Reader the agents still work — you just lose local-library retrieval

### 4. Never commit keys or local paths

The bundled `cordis.patch.yml` contains no API keys and no absolute paths. The skills directory and Reader location are resolved from environment variables.

## Install

### Option 1: local directory (best for development)

```bash
git clone https://github.com/chang30519-art/dsh-ars-agents.git
```

Then in DSH **Plugins → Add plugin**, enter the cloned **absolute local path**, for example:

```
E:\GitHub\dsh-ars-agents
```

### Option 2: git URL

In **Plugins → Add plugin**, enter:

```
https://github.com/chang30519-art/dsh-ars-agents
```

### Option 3: merge the patch by hand

If you would rather not install a plugin, append the contents of `cordis.patch.yml` to:

```
<DSH_HOME>/profiles/desktop/cordis.patch.yml
```

Back it up first. Restart DSH afterwards.

> DSH rewrites profile files on restart, so hand-merged changes can be overwritten. The plugin route is recommended.

## Environment variables

| Variable | Purpose | Default |
|---|---|---|
| `DSH_HOME` | DSH data directory | `%USERPROFILE%\.dsh` |
| `DSH_ARSS_SKILLS_DIR` | Skills parent directory (overrides default) | `<DSH_HOME>/skills` |
| `READER_CLI` | Full path to `reader.exe` | resolved from PATH |

## Verify the install

1. Restart DSH
2. Open a new session — the preset picker should list **ARS·Main / Research / Screener / Writer / Reviewer / Verify / Experiment**
3. Pick **ARS·Main** and ask "what does my library say about X?" — it should `skill`-load `academic-research-suite` first, then run `reader search`

If the presets do not appear, check:

- the bundle shows **On** in the Plugins page
- `<DSH_HOME>/skills/academic-research-suite` exists
- the startup log for `cannot resolve` style module errors

## Architecture

```
┌─────────────────────────────────────────┐
│  ARS·Main (ars-pipeline, order 1)        │
│  10-stage orchestration + dispatch tool  │
└──────────────┬──────────────────────────┘
               │ dispatch (allow-listed)
   ┌───────────┼───────────┬───────────┐
   ▼           ▼           ▼           ▼
Research    Screener    Writer     Reviewer
   │           │           │           │
   └───────────┴─────┬─────┴───────────┘
                     ▼
                  Verify
                     │
                     ▼
                 Experiment
```

Each specialist's persona carries the real content of its workflow — phases, gates, mode tables, output contracts — rather than pointing at a document to read. Only the deep detail falls back to `references/` and `templates/` inside the skill pack.

## Known limitations

- **Shell tool is platform-split**: `tool-pwsh` on Windows, `tool-bash` elsewhere, both gated by `process.platform`. Windows is the best-tested path.
- **Personas are in Chinese**: these agents reason and report in Chinese, switching to English for manuscript prose when asked. The instruction layer is Chinese.
- **Tied to the skill pack layout**: if upstream `academic-research-skills` changes its directory layout (e.g. renames `ars/<workflow>/WORKFLOW.md`), the persona path references need updating.
- **No WeKnora integration**: this package only uses the Reader CLI for local retrieval. If you use WeKnora, install that plugin and add its rows yourself.

## Licence and attribution

- Code and persona text in this package: **MIT** (see [LICENSE](LICENSE))
- `academic-research-suite` skill pack: **CC-BY-NC-4.0**, by Cheng-I Wu, **not bundled here**. Using, modifying, or redistributing that pack requires independently complying with its attribution and non-commercial terms.

The personas reference and adapt that pack's workflow structure and role naming. If you redistribute this package together with that pack, you must also satisfy CC-BY-NC-4.0.
