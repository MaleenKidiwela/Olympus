# Olympus — a multi-agent family for Claude Code

Olympus is a coordinated team of subagents for Claude Code. An orchestrator (**Zeus**)
decomposes a goal, assigns each piece to the specialist whose nature fits it, routes
finished work to an independent adversarial reviewer, and stops at a human gate before
anything consequential or irreversible. This repository is both a **plugin** and a
**plugin marketplace**, so you can install the whole family on any machine with two
commands.

## The family (14 agents)

| Agent | Role | Model |
|-------|------|-------|
| **zeus** | Orchestrator — decomposes the goal, assigns specialists, runs the loop, gates to the human | opus |
| **athena** | Planner — phased plans, trade-offs, a completeness check (read-only) | opus |
| **atlas** | Adversarial auditor — finds flaws *and* omissions; never writes or runs | opus |
| **apollo** | Visionary — surfaces unexplored angles/experiments, grounded; checked by Atlas (read-only) | opus |
| **prometheus** | Researcher — surveys prior art & standard practice; briefs with sources | opus |
| **metis** | Resource surveyor — measures the *effective* CPU/mem/GPU limits (cgroup/pod/SLURM, not nominal) and budgets the build | sonnet |
| **hephaestus** | Builder — writes/edits code, verifies with real checks | opus |
| **theseus** | Debugger — follows the thread to root cause and fixes at the source | opus |
| **cassandra** | QA/testing — writes & runs tests, probes edges, ends with a PASS/FAIL verdict | sonnet |
| **argus** | Guardian — runtime watch (crash/stall/OOM) + static security review | opus |
| **daedalus** | Frontend/design — the user-facing visual layer | opus |
| **calliope** | Writer/chronicler — docs, method/results notes, daily log | sonnet |
| **hermes** | Courier — fast mechanical work (renames, moves, formatting) | haiku |
| **mnemosyne** | Keeper of the record — commits & pushes to git | sonnet |

## Install (any machine)

```shell
# Add this marketplace (use your GitHub org/repo once it's pushed, or a local path)
/plugin marketplace add <owner>/olympus-marketplace
# or from a local clone:
/plugin marketplace add ./olympus-marketplace

# Install the family
/plugin install olympus@olympus
```

Updates: `/plugin marketplace update olympus`, then `/plugin install olympus@olympus` again.

## Run it

The plugin ships an `/olympus` command that kicks off the whole team on a goal:

```shell
/olympus <your goal>
```

This primes Claude as **Zeus** and runs the family's loop — plan (Athena) → audit (Atlas)
→ research (Prometheus) / resource-budget (Metis) → build (Hephaestus) + test (Cassandra)
+ watch (Argus) → adversarial verify (Atlas) → document (Calliope) → commit only on your
approval (Mnemosyne) — stopping at a human gate before anything consequential. (Plugin
commands are namespaced, so depending on your version the slash may appear as
`/olympus:olympus`; check the `/` command picker.)

You can also invoke any specialist directly by its `subagent_type` (e.g. `zeus`, `atlas`,
`apollo`), or just describe a goal and let your session route to Zeus.

Team auto-onboarding: add this to a shared project `.claude/settings.json` so teammates
are prompted to install on clone:

```json
{
  "extraKnownMarketplaces": {
    "olympus": { "source": { "source": "github", "repo": "<owner>/olympus-marketplace" } }
  },
  "enabledPlugins": { "olympus@olympus": true }
}
```

## Alternative: copy the files (needed if you add per-agent hooks)

Plugin-delivered agents support `tools`, `model`, and the prompt body, but **ignore**
the per-agent frontmatter fields `hooks`, `mcpServers`, and `permissionMode`. The
current Olympus agents use none of those, so the plugin install is fully featured. If
you later add a per-agent `hooks` block (e.g. a `Stop`-hook auto-verify loop) to an
agent, install *that* agent by copying its file into `.claude/agents/` (project scope)
or `~/.claude/agents/` (user scope) — a same-named file overrides the plugin version.

## Layout

```
olympus-marketplace/
├── .claude-plugin/
│   └── marketplace.json        # the catalog (marketplace name: "olympus")
└── plugins/
    └── olympus/
        ├── .claude-plugin/
        │   └── plugin.json     # the plugin manifest (plugin name: "olympus")
        └── agents/             # the 14 agent definitions (auto-discovered)
            └── *.md
```

## Validate before publishing

```shell
claude plugin validate ./plugins/olympus --strict
```

## How they work together

A typical run: **athena** plans → **atlas** reviews it for flaws *and* missing standard
components → human approves → **prometheus** confirms method/library details and
**metis** budgets the compute → **hephaestus** builds → **cassandra** tests and
**argus** watches the run → **atlas** audits the result → **apollo** proposes
unexplored follow-ups (re-checked by atlas) → **calliope** documents → **mnemosyne**
commits (only on human approval). Zeus drives this as a bounded maker→critic→fix loop
and never ships past a Critical finding by fiat.
