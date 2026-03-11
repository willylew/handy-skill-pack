# Handy Skill Pack

Handy Skill Pack is a small collection of handy dandy AI coding skills.

It is intentionally pretty narrow. These are not broad framework tutorials. They are targeted, practical skills for production work.

This repo is packaged in two ways:

- **Claude Code**: as a plugin marketplace with installable plugins
- **Codex CLI**: as individual installable skills inside the repo

## Included Skills

The repository currently includes two Claude plugins:

- `temporal-python-skills`
- `kafka-reliability-skills`

Across those plugins, the included skills are:

| Skill | Plugin | Purpose |
| --- | --- | --- |
| `temporal-retry-policies` | `temporal-python-skills` | Design Temporal Activity timeouts and retry policies for outbound HTTP and service calls in the Python SDK. |
| `temporal-testing-ci` | `temporal-python-skills` | Test Temporal Python workflows with time skipping and replay in CI to catch nondeterminism and logic bugs early. |
| `temporal-versioning-determinism` | `temporal-python-skills` | Evolve Temporal Python workflows safely across deployments using patching, worker versioning, and replay tests. |
| `kafka-reliability-idempotency` | `kafka-reliability-skills` | Build reliable classic Kafka consumers and producers with explicit commit strategy, idempotent processing, and DLQ handling. |

## Use With Claude Code

Add this repository as a marketplace and install whichever plugin you want:

```bash
claude plugin marketplace add willylew/handy-skill-pack
claude plugin install temporal-python-skills@handy-skill-pack
claude plugin install kafka-reliability-skills@handy-skill-pack
```

To pull later updates:

```bash
claude plugin marketplace update handy-skill-pack
claude plugin update temporal-python-skills
claude plugin update kafka-reliability-skills
```

## Use With Codex CLI

Codex installs these as individual skills rather than as one plugin.

### Repo-local install (`./.agents/skills`)

For team-shared or repo-specific Codex usage, install the skills into the current repository:

```bash
mkdir -p ./.agents/skills

python "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo willylew/handy-skill-pack \
  --path plugins/temporal-python-skills/skills/temporal-retry-policies \
         plugins/temporal-python-skills/skills/temporal-testing-ci \
         plugins/temporal-python-skills/skills/temporal-versioning-determinism \
         plugins/kafka-reliability-skills/skills/kafka-reliability-idempotency \
  --dest ./.agents/skills
```

This creates skill folders under `./.agents/skills/` in your repo.

### User-level install

If you want the skills available across all repos for one user, install them from inside a Codex session by pointing the built-in skill installer at the GitHub directory:

```text
$skill-installer install https://github.com/willylew/handy-skill-pack/tree/main/plugins/temporal-python-skills/skills/temporal-retry-policies
$skill-installer install https://github.com/willylew/handy-skill-pack/tree/main/plugins/temporal-python-skills/skills/temporal-testing-ci
$skill-installer install https://github.com/willylew/handy-skill-pack/tree/main/plugins/temporal-python-skills/skills/temporal-versioning-determinism
$skill-installer install https://github.com/willylew/handy-skill-pack/tree/main/plugins/kafka-reliability-skills/skills/kafka-reliability-idempotency
```
