# State of AI Instruction Quality, Q3 2026: companion dataset

Dataset for "The State of AI Instruction Quality, Q3 2026", a measurement of the agent instruction files in 10,000 high-star English GitHub repositories (data: September 2026 snapshot). Measured with [Reporails CLI](https://github.com/reporails/cli). The Q2 2026 dataset (28,721 repositories, April 2026) is in `../2026-Q2/`.

## What's in this dataset

| File                | Records | Description                                                             |
|---------------------|--------:|-------------------------------------------------------------------------|
| `repos.jsonl`       |  10,000 | Per-repository: agents detected, stars, language, archived flag          |
| `stats_public.json` |       1 | Aggregate statistics backing the report's tables and headline figures    |

## Data dictionary

### repos.jsonl

| Field              | Type      | Description                                                                                          |
|--------------------|-----------|------------------------------------------------------------------------------------------------------|
| `full_name`        | string    | `owner/repo`                                                                                         |
| `canonical_agents` | string[]  | Agents with a detected setup: `claude`, `generic` (`AGENTS.md`), `copilot`, `cursor`, `antigravity`, `codex`. Empty for the 216 repos with none |
| `stars`            | int       | GitHub stars at collection (September 2026)                                                          |
| `primary_language` | string?   | GitHub-detected primary language                                                                     |
| `license`          | null      | Not collected for this release                                                                       |
| `archived`         | bool?     | Archived flag from the 2026-09-02 metadata refresh (null where not collected)                         |
| `topics`           | null      | Not collected for this release                                                                       |

### stats_public.json

Same section layout as the Q2 file, plus `report_figures`. Definitions that differ from Q2 are listed under [Definitions](#definitions).

| Section              | What it contains                                                                 |
|----------------------|----------------------------------------------------------------------------------|
| `headline`           | Repos, agent setups, files, atoms, charged atoms, diagnostics                     |
| `agent_distribution` | Setups per agent, agents-per-repo histogram, co-occurrence pairs                   |
| `band_distribution`  | Score bands (LOW below 5, MODERATE 5 to under 7, HIGH 7 and up) globally and per agent |
| `top_rules`          | Most common diagnostics, with setups and repos affected                           |
| `specificity`        | Named vs abstract share of instructions (charged atoms), per agent                |
| `multi_agent_delta`  | Within-repo best and worst agent counts and band-spread counts (no repo names)    |
| `file_count_quality` | Score bands by instruction-file count                                             |
| `config_types`       | Bands, named share and terse share by file type (main, rules, skills, sub-agents, other) |
| `content_length`     | Median atoms, directives and tokens per file type                                 |
| `descriptive`        | File sizes, directive ratio, instruction length, most common file paths           |
| `skills_quality`     | Most-adopted skills with bands and named share                                    |
| `subagents_quality`  | Most-adopted sub-agent roles with named share                                     |
| `report_figures`     | The report's headline figures with their denominators                             |

## Definitions

- **Agent setup:** one agent's instruction files in one repository. A repository with `CLAUDE.md` and `AGENTS.md` has two setups (`claude` and `generic`).
- **Agents:** `claude`, `generic` (`AGENTS.md`), `copilot`, `cursor`, `antigravity` (Gemini's agent), `codex` (repositories with a `.codex/` folder). Q2 used `gemini` and counted `AGENTS.md` as `codex`.
- **Atom:** one instruction unit after parsing. `content_atoms` excludes headings. A charged atom is a directive or a constraint; a named atom names a specific tool, file or command.
- **Score bands:** LOW below 5, MODERATE 5 to under 7, HIGH 7 and up, on the 0 to 10 score `ails check` reports. Q2's bands came from a different scorer and are not comparable.
- **Coverage:** most sections cover the 15,219 setups with diagnostics; `specificity` covers all 15,237 measured setups (`specificity.setups_covered`). So `headline.total_charged` (1,894,268 directives and constraints, the denominator of the report's per-instruction rate) and `specificity.total_charged` (1,954,750) count different setup populations. Per-agent setup counts are up to 18 below the report's counts for the same reason.
- **Rules:** `top_rules` counts `repos_affected` as distinct repositories (Q2 counted setups under that name; `setups_affected` keeps the setup count).
- **File types:** main, rules, skills, sub-agents, and `other` (commands and prompts), by each agent's documented folder conventions. The top file-count bucket is 51 to 90 files.

## Methodology

### Sampling frame

- 10,000 public GitHub repositories carrying at least one instruction file of a supported coding agent (Claude Code, Codex, GitHub Copilot, Cursor, Gemini, or a shared `AGENTS.md`), selected top-down by stars, then filtered to English instruction files.
- Stars: minimum 493, median 1,551, maximum 454,882. Findings about stars hold within this range, not for small repositories.
- English gate: a deterministic script check (no model) over every instruction file; 8.2% of the hunted repositories were excluded as non-English.
- Snapshot: each repository was cloned in September 2026. Files changed since then are not reflected.

### Measurement

- Each agent's instruction files at the default locations and exact names its documentation specifies (`CLAUDE.md`, `AGENTS.md`, rules, skills, sub-agents, commands), mapped and scored by the same `ails check` pipeline anyone can run.
- Instrument, frozen for the whole run: Reporails CLI 0.6.1 for parsing and 0.6.2 for diagnostics and scoring.
- On 2026-10-13, 7,044 files in folders the original clone step had missed (native sub-agent folders, commands, prompts and hook folders) were fetched at each repository's September commit and measured on the same instrument.
- 15,237 agent setups in 9,784 repositories were measured; 15,219 have diagnostics, 14,961 have a score. 258 setups hold no directive or constraint and receive no score.
- 158 of the largest setups (more than 90 instruction files each, about 1% of setups) were still being measured at publication and are not in these files. They will be added as an update.

### Known biases

- GitHub only, public repositories only.
- Star-first selection: high-star repositories only (493 stars and up).
- English instruction files only.
- Default load locations only: docs an agent opens only when linked are outside the measurement.
- Snapshot: instruction practices change quickly.
- Measured with our own scorer. Run it on your repository and compare.

## How to verify report claims

### "10,000 repositories"
```bash
wc -l repos.jsonl
```

### "43.1% of repositories configure more than one agent"
```bash
python3 -c "
import json
r=[json.loads(l) for l in open('repos.jsonl')]
w=[x for x in r if x['canonical_agents']]
m=sum(1 for x in w if len(x['canonical_agents'])>1)
print(f'{m}/{len(w)} = {100*m/len(w):.1f}%')
"
```

### "91.1% of repositories carry at least one vague instruction"
Read `report_figures.vague_instruction_prevalence` in `stats_public.json` (8,910 of 9,776 repositories with a linted setup).

### Independent reproduction
Clone any repository from `repos.jsonl` and run `uvx --from reporails-cli ails check` on it.

## What's not in this dataset

- **Instruction file content:** the text belongs to the repository owners.
- **Per-repository and per-file scores:** only bands and aggregates are published.
- **Calibration constants and embeddings.**

## License

`repos.jsonl` references public GitHub repositories. `stats_public.json` is original analysis by [Reporails](https://github.com/reporails/cli). CC-BY-4.0.
