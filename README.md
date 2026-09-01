# Sub Agents Chain

A set of subagents for Claude Code, organised as a chain: one agent takes the task and talks to
you, leads split the work, workers carry it out, and separate agents check the result.

The set covers analysis, specification, edits and audit. Each stage ends with a report; where one
stage feeds into another, it ends with a question, and the next stage starts on your answer.

**Built for effectiveness, not for token economy.** The chain is designed to arrive at a correct
result, and it spends tokens to get there. Where a choice existed between a cheaper run and a
better-established one, the better-established one was taken.

**The Builder Team, the Audit Team and the Planner are still under development.** That is the
Execution group (`3-execution/`), the Control group (`4-control/`) and `planner`. They work, and
they are still being improved — expect them to change more than the rest of the set.

---

## The agents

**Main**

| Agent | Role |
|---|---|
| `conductor` | Takes the task, decides which group it belongs to, launches the agents a lead names, and reports back to you |
| `what-if` | Reviews conclusions before they are acted on |
| `deep-analyst` | Reads the reports of a run together and returns one consolidated reading |
| `planner` | Turns an approved outcome into a specification |

**Analysis** — `2-analysis/`

| Agent | Role |
|---|---|
| `analysis-lead` | Splits a question into parts, writes the assignments, judges the reports |
| `analyst` | Examines the code: how it works, what depends on it, and the available options |
| `scout` | Researches sources outside the repository |

**Execution** — `3-execution/`

| Agent | Role |
|---|---|
| `build-lead` | Splits a specification into change units and assigns them |
| `rust-builder` · `ts-builder` · `python-builder` | Code changes in the respective language |
| `builder` | Changes to non-code files — documents, configs, texts |
| `refactorer` | Restructuring without changing behaviour |
| `sweeper` | The same edit applied across many places |
| `test-writer` | Writes tests |
| `cleaner` | Removes archive copies, on request and with a file list |

**Control** — `4-control/`

| Agent | Role |
|---|---|
| `audit-lead` | Decides what to check after a change and assigns it |
| `auditor` | General review of changed code |
| `security-auditor` · `concurrency-auditor` · `perf-auditor` · `docs-drift-auditor` | Narrow reviews: security, concurrency, performance, documentation against behaviour |
| `architect-reviewer` | Compares the implementation with the recorded architecture |
| `integrator` | Checks the seams between separately built parts |
| `debugger` | Locates a defect and identifies its cause |
| `tester` | Runs tests and reports the result |

**Independent** — `1-independent/`

| Agent | Role |
|---|---|
| `judge` | Checks that the standing rules were followed and the instruction carried out |
| `detective` | Checks what was actually changed on disk against what was permitted |

## How a run flows

```
   you
    │
    ▼
 conductor ──► lead ──► workers ──► lead ──► checkers ──► conductor
    │                                                          │
    └──────────────────────── report ◄─────────────────────────┘
```

Agents do not address one another. The conductor passes paths between stages, and each report
reaches the next stage in the form it was written.

## Stages

| Stage | Input | Output |
|---|---|---|
| Analysis | a question | the available options and what each involves |
| Specification | an approved outcome | a specification for the execution group |
| Edits | an approved specification | the change, with a record of what was touched |
| Audit | a request to check | findings, each with its location |

Each stage ends with a report to you.

---

## Install

Copy the five group folders into your agents directory:

```
~/.claude/agents/               available in every project
<your-project>/.claude/agents/  available in one project
```

```
Main/  1-independent/  2-analysis/  3-execution/  4-control/
```

The directory is scanned recursively; an agent's name comes from the `name` field in its own
file, so the folder names are for your convenience.

### Set the paths

The agent files refer to their configuration by absolute path, with the machine-specific segments
written in angle brackets. Replace each with your own value — the same value everywhere it
appears:

| Placeholder | Value |
|---|---|
| `<username>` | your user name |
| `<project-slug>` | the folder Claude Code keeps for the project under `~/.claude/projects/`, named after the project's absolute path with the separators replaced by dashes |
| `<workspace>` | where your reports and specifications are kept |
| `<project>` | the project being worked on |

A find-and-replace across the folder covers all four.

### Two configuration files

| File | Contents |
|---|---|
| `rules.md` | The standing rules the agents read before acting |
| `PROJECT-BRIEF.md` | The project's own rules: layout, invariants, build and test commands |

Both ship filled in as working examples. Replace them with your own.

Two conventions to keep when rewriting `rules.md`: the rules are numbered and referred to by
number, and two of its sections are referred to by name — the one describing the parts of an
assignment, and the one describing where files are kept.

A project may override either file: if `.claude/rules.md` or `.claude/PROJECT-BRIEF.md` exists in
the project being worked on, it takes precedence.

### Where the chain writes

```
.claude/<chain-name>/reports/     reports for you
.claude/<chain-name>/runs/        what agents write to each other
.claude/<chain-name>/specs/       specifications
```

---

## Use

Run the whole chain with the conductor as the session's main agent:

```
claude --agent conductor
```

Or call a single agent directly:

```
@agent-analyst     what does this module do, and what depends on it?
@agent-scout       what exists for this outside the repository?
@agent-tester      run the tests and report the result
```

## Requirements

- **Claude Code** with subagent support and the `--agent` flag.
- **A shell** for the agents that declare one. Most declare both `PowerShell` and `Bash`; on
  macOS or Linux you can remove `PowerShell` from the `tools` line where it appears.
- **Network access** for the agents that declare `WebSearch` and `WebFetch`.

The model each agent asks for is set in its own frontmatter and can be changed there; some agents
also set a reasoning effort. A run dispatches several subagents, most of them on Opus — worth
checking your usage settings before the first full run.

**Name collisions.** The agent names are common words. If an agent of the same name already
exists in `~/.claude/agents/` or the project's `.claude/agents/`, that one takes precedence and
this one is not loaded.

## License

MIT — see `LICENSE`.
