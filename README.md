# 30K Corpus: AI Instruction Quality Dataset

Dataset for ["The State of AI Instruction Quality"](https://medium.com/@cleverhoods/the-state-of-ai-instruction-quality) — a corpus analysis of 28,721 GitHub repositories containing instruction files for five coding agents.

## What's in this dataset

| File                 | Records | Description                                                      |
|----------------------|--------:|------------------------------------------------------------------|
| `repos.jsonl`        |  28,721 | Per-project: agents configured, stars, language, license, topics |
| `stats_public.json`  |       1 | Aggregate statistics backing every table in the article          |
| `validation_key.csv` |   2,814 | Sample classifications: atom text, tool labels, source location  |

## Data dictionary

### repos.jsonl

Each line is a JSON object with:

| Field              | Type     | Description                                                                           |
|--------------------|----------|---------------------------------------------------------------------------------------|
| `full_name`        | string   | `owner/project`                                                                       |
| `canonical_agents` | string[] | Agents detected after deduplication: `claude`, `codex`, `copilot`, `cursor`, `gemini` |
| `stars`            | int      | GitHub star count at collection time                                                  |
| `primary_language` | string?  | GitHub-detected primary language                                                      |
| `license`          | string?  | SPDX license identifier                                                               |
| `archived`         | bool     | Whether repo is archived                                                              |
| `topics`           | string[] | GitHub topics                                                                         |

### stats_public.json

Aggregate statistics organized by section:

| Section              | What it contains                                                                           | Article reference        |
|----------------------|--------------------------------------------------------------------------------------------|--------------------------|
| `headline`           | Total projects, files, atoms, charged atoms                                                | Headline numbers         |
| `agent_distribution` | Per-agent project counts, multi-agent histogram, co-occurrence pairs                       | Dataset table, Finding 5 |
| `band_distribution`  | Compliance band distribution (LOW/MODERATE/HIGH) per agent                                 | —                        |
| `specificity`        | Named vs abstract atom counts, per-agent breakdown                                         | Finding 2                |
| `config_types`       | File counts and specificity by config type (base/rules/skills/agents), per-agent breakdown | Finding 4, Anatomy chart |
| `content_length`     | Median atoms, directives, tokens per config type                                           | Finding 1, Anatomy chart |
| `descriptive`        | File size, directive ratio, instruction length, top file names                             | Finding 1, Finding 3     |
| `skills_quality`     | Per-skill adoption and specificity (named%) for top 30 skills                              | Finding 6                |
| `subagents_quality`  | Per-role specificity for top 15 sub-agent roles                                            | Finding 7                |
| `top_rules`          | Most common diagnostic categories across corpus                                            | —                        |
| `multi_agent_delta`  | Multi-agent project counts, per-agent win/loss counts                                      | Finding 5                |
| `file_count_quality` | Quality band distribution by instruction file count                                        | —                        |

### validation_key.csv

Sample classifications from the corpus across star tiers (top-100, mid-tier, long-tail repos). Each row shows the tool's classification alongside the source text for inspection.

| Field              | Type   | Description                                                        |
|--------------------|--------|--------------------------------------------------------------------|
| `id`               | int    | Sample identifier                                                  |
| `project`          | string | Repository name                                                    |
| `agent`            | string | Agent configuration type                                           |
| `file_path`        | string | Path within the repository                                         |
| `line`             | int    | Line number in the source file                                     |
| `text`             | string | The instruction text being classified                              |
| `tool_charge`      | string | Tool classification: CONSTRAINT, DIRECTIVE, IMPERATIVE, or NEUTRAL |
| `tool_modality`    | string | Detection method: direct, imperative, or none                      |
| `tool_specificity` | string | Named (references specific construct) or abstract (category-level) |

## Sampling methodology

### Collection

1. **Source**: GitHub REST API search, querying for repositories containing instruction files for five coding agents (CLAUDE.md, agents.md, .cursorrules, copilot-instructions.md, gemini.md, and variants)
2. **Deduplication**: Agent detection overlap resolved per project — a file matching patterns for multiple agents is assigned to the most specific match
3. **Window**: Collected March-April 2026
4. **Total**: 28,721 projects with analyzable instruction content after deduplication and filtering

### Analysis

Each instruction file was processed by the [Reporails](https://github.com/reporails/cli) built-in classifier:
- **Parsing**: Markdown to atomic units (one semantic chunk = one atom)
- **Charge classification**: Three-phase deterministic pipeline (negation detection, modal auxiliary detection, syntactic dependency parsing). No LLM. First definitive match wins.
- **Specificity**: Binary — does the instruction name a specific construct (tool, file, command, flag) or stay at the category level?
- **File categorization**: Base config, rules, skills, or sub-agents — based on file path conventions per agent

The classifier is source-available ([BUSL-1.1](https://github.com/reporails/cli/blob/main/LICENSE)). Run `npx @reporails/cli check` on any project to inspect its diagnostics.

### Known biases

- **GitHub-only**: No GitLab, Bitbucket, or private repos
- **English-skewed**: GitHub API search favors English-language repositories
- **Public-only**: Enterprise and private configurations are not represented
- **Activity bias**: Repos must have enough activity to appear in GitHub search results
- **No popularity weighting**: A 10-star hobby project counts the same as a 50K-star production repo
- **Snapshot**: Instruction practices are changing rapidly. These numbers describe the ecosystem at collection time.

### Classification approach

Charge classification uses a three-phase deterministic pipeline (negation detection → modal auxiliary detection → syntactic dependency parsing). No LLM. Specificity detection (named vs abstract) uses backtick/code-token patterns and is simpler and more robust. The `validation_key.csv` provides sample classifications for inspection.

## How to verify article claims

### "28,721 repositories"
```bash
cat repos.jsonl | wc -l
```

### "43% Claude"
```bash
cat repos.jsonl | python3 -c "
import sys, json
repos = [json.loads(l) for l in sys.stdin]
claude = sum(1 for r in repos if 'claude' in r['canonical_agents'])
print(f'{claude}/{len(repos)} = {claude/len(repos)*100:.1f}%')
"
```

### "37% configure multiple agents"
```bash
cat repos.jsonl | python3 -c "
import sys, json
repos = [json.loads(l) for l in sys.stdin]
multi = sum(1 for r in repos if len(r['canonical_agents']) > 1)
print(f'{multi}/{len(repos)} = {multi/len(repos)*100:.1f}%')
"
```

### All other statistics
Compare any number in the article against the corresponding field in `stats_public.json`. Every table in the article traces to a specific section in this file.

### Independent reproduction
Clone any subset of repos from `repos.jsonl`, run `npx @reporails/cli check` on each, and compare your diagnostics against ours.

## What's not in this dataset

- **Instruction file content**: The actual text of instruction files belongs to the repo owners. This dataset contains only structural metadata and aggregate statistics.
- **Continuous quality scores**: Per-file and per-project quality scores are computed server-side using proprietary calibration constants and are not included. Compliance bands (LOW/MODERATE/HIGH) are included as they do not expose calibration values.
- **Embeddings**: Vector representations of instruction content are not included.

## License

The repository index (`repos.jsonl`) references public GitHub repositories. The aggregate statistics (`stats_public.json`) and validation sample (`validation_key.csv`) are original analysis by [Reporails](https://github.com/reporails/cli).

CC-BY-4.0
