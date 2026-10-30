**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥16-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/repodoctor)](LICENSE)
<!-- /badges -->

# repodoctor

A zero-dependency, single-file **Git repo hygiene CLI**. One command scans your git repo and produces a 0–100 health score with an actionable fix list, ready to drop into a CI gate.

`devdoctor` only scans source-layer plaintext secrets; `repodoctor` adds the **git-layer** engineering hygiene — the "9th axis" beyond the code-health family.

## Who should use it / Who shouldn't

- **Use it if** you just inherited a repo and want to quickly surface `.env` leaks / large files / missing LICENSE; you want a pre-release check-up for an open-source project; or you want a repo-hygiene gate in CI.
- **Skip it if** you already run gitleaks + git-secrets with solid secret coverage (it still works as a complement); or your repo is purely local, never published, and doesn't care about LICENSE.

## Why it's needed

- `.env` / `node_modules` accidentally committed (secret leak, repo bloat)
- Plaintext secrets once written into a commit linger in git history even after deletion
- Large files committed directly, repo keeps growing (should use Git LFS)
- Open-source project missing LICENSE / README, or dependencies without a lockfile (not reproducible)
- Commit messages all `update` / `fix` / `wip` — history is untraceable

Tools like gitleaks (needs install + heavy rules) and git-secrets (needs hooks) can't scan out-of-the-box with zero dependencies, and cover a narrower set of dimensions. repodoctor checks it all in one command.

## Install / Run

No npm install — just Node (>=16) and git:

```bash
node repodoctor.js              # scan the repo in the current dir
node repodoctor.js /path/repo  # scan a specific repo
node repodoctor.js --help      # show all options
```

## Detection dimensions

| Rule | Dimension | Severity | Description |
|------|----------|----------|-------------|
| G1 | dangerous files / ignore | HIGH / MED | `.env`, node_modules, dist etc. committed; missing `.gitignore` |
| G2 | secret leak | HIGH | Plaintext secrets / credentials in current file content + recent N commit diffs |
| G3 | large files | HIGH | Files over a threshold (default 100MB) committed to the repo |
| G4 | LICENSE | MED | Missing LICENSE file |
| G5 | README | MED | Missing README file |
| G6 | lockfile | MED | Has package.json but no dependency lockfile |
| G7 | commit hygiene | LOW | Too high a ratio of meaningless commits (update/fix/wip…) |
| G8 | branch hygiene | LOW | Too many local + remote branches |

## Health score

`health = max(0, 100 − HIGH×12 − MED×6 − LOW×3)`. HIGH issues (secret leak / large files) deduct heavily — intuitively.

## CI gate

```bash
# Any HIGH issue → exit code 1
node repodoctor.js --fail-on-high

# Health score below 90 → exit code 1
node repodoctor.js --min-score 90
```

GitHub Actions example:

```yaml
- name: repo hygiene check
  run: node repodoctor.js --fail-on-high --min-score 90
```

## Options

- `-r, --root <dir>` target repo (default: current dir)
- `--max-size <MB>` large-file threshold, default 100
- `--history-depth <n>` scan diffs of the last n commits, default 100
- `--min-score <n>` exit 1 if health score below n
- `--fail-on-high` exit 1 if any HIGH exists
- `--json` output JSON
- `-q, --quiet` output summary only
- `-V, --version` version number

## Zero-dependency note

Uses only Node built-in modules plus the system `git` binary (the natural prerequisite for repo check-ups). No network, never executes any code in the scanned repo — pure read-only analysis.

## Roadmap

Rule set is stable. Possible future additions: baseline/allowlist support for known-large assets and a secret-allowlist. No breaking changes planned.

## Contributing

Issues and pull requests are welcome. Keep it single-file and zero-dependency.

## ⭐ Star

If repodoctor ever stopped an `.env` misfire or a secret leak for you, a Star ⭐ helps others find it: https://github.com/huanweide/repodoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
