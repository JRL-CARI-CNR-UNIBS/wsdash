# wsdash — `cbws` and `gitws`

Two terminal (curses) tools for working in a ROS 2 workspace:

| Tool     | What it does                                                        |
|----------|---------------------------------------------------------------------|
| `cbws`   | Interactive `colcon build` manager: status of every package, build/clean selections, view logs |
| `gitws`  | Overview of all git repositories under a directory: status, stage, commit, pull, push |
| `cbfail` | Build-log viewer (errors/warnings per package). Used by `cbws`, also works on its own |

All three are single-file Python 3 scripts using only the standard library.

## Install

Download the scripts and copy them into `~/.local/bin`:

```bash
BASE=https://github.com/JRL-CARI-CNR-UNIBS/wsdash/bin
cd /tmp
wget https://raw.githubusercontent.com/JRL-CARI-CNR-UNIBS/wsdash/refs/heads/master/bin/cbws
wget https://raw.githubusercontent.com/JRL-CARI-CNR-UNIBS/wsdash/refs/heads/master/bin/cbfail
wget https://raw.githubusercontent.com/JRL-CARI-CNR-UNIBS/wsdash/refs/heads/master/bin/gitws
mkdir -p ~/.local/bin
cp cbws gitws cbfail ~/.local/bin/
chmod +x ~/.local/bin/cbws ~/.local/bin/gitws ~/.local/bin/cbfail
rm cbws gitws cbfail
```

From a local copy of this folder, just:

```bash
mkdir -p ~/.local/bin
cp bin/cbws bin/gitws bin/cbfail ~/.local/bin/
chmod +x ~/.local/bin/cbws ~/.local/bin/gitws ~/.local/bin/cbfail
```

Make sure `~/.local/bin` is in your `PATH` (`export PATH="$HOME/.local/bin:$PATH"` in `~/.bashrc`).

**Requirements:** Python 3, `git` (for `gitws`), `colcon` and a sourced ROS 2 environment (for `cbws`).
Keep `cbfail` in the same directory as `cbws` (or anywhere in `PATH`), otherwise the log viewer
in `cbws` is disabled.

---

## cbws — colcon build manager

```bash
cbws            # interactive interface
cbws -s         # print a status table and exit
cbws -w ~/ws    # use a specific workspace
```

### How it works

- **Finding the workspace:** starting from the current directory, it walks up until it finds a
  directory with `src/` and at least one of `build/`, `install/`, `log/`. So you can launch it
  from anywhere inside the workspace.
- **Package list and order:** read with `colcon list` and `colcon graph` (run with
  `--log-base /dev/null`, so they don't add entries to `log/`).
- **Build status:** taken from the colcon logs in `log/build_*` (the last ~60 builds). For each
  package it shows the last result (`OK`, `FAILED`, `ABORTED`, `SKIPPED`, `never`), when it was
  built, the build type, and the number of errors/warnings found in its log.
- **"changed" flag:** a package is marked *changed* when a file in its source tree is newer than
  its last successful build.
- **External builds:** if a `colcon build` started from another terminal is running in the same
  workspace, cbws shows it and refreshes when it ends.
- **Building:** cbws leaves the interface, runs `colcon build` in the terminal (so you see the
  normal output), prints a short summary, then returns to the list.

The build command is:

```
colcon build --event-handlers console_cohesion+ [--symlink-install] [--continue-on-error]
             [--parallel-workers N] [--cmake-clean-cache] [--cmake-force-configure]
             <selection> <extra colcon args>
             --cmake-args [-DCMAKE_EXPORT_COMPILE_COMMANDS=ON] [-DCMAKE_BUILD_TYPE=...] <extra cmake args>
```

Options are set with `O` and saved **per workspace** in `~/.config/cbws/settings.json`.
Defaults: symlink install ON, continue-on-error ON, compile_commands ON, jobs auto, build type
not set.

### Keys (package list)

Actions apply to the **marked** packages (SPACE) if any, otherwise to the selected one.

| Key | Action |
|-----|--------|
| `↑↓ j k PgUp PgDn g G` | move |
| `ENTER` | action menu for the package |
| `SPACE` / `M` | mark / clear all marks |
| `b` | build only these (`--packages-select`) |
| `u` | build up to these, with their dependencies (`--packages-up-to`) |
| `a` | build these and everything that depends on them (`--packages-above`) |
| `A` | like `a`, plus all their dependencies |
| `B` | build the whole workspace |
| `F` | rebuild the packages that failed in the last build |
| `n` | build the *changed* packages and their dependents |
| `R` | clean rebuild (delete `build/<pkg>`, `install/<pkg>`, then build) |
| `t` | cycle build type: Default / Debug / Release / RelWithDebInfo / MinSizeRel |
| `O` | build options |
| `x` / `X` | clean selected / clean whole workspace (`build/`, `install/`, optionally `log/`) |
| `l` | errors and warnings of the package's last build |
| `L` | browse the last builds with cbfail |
| `e` | jump to the next failed package |
| `d` | dependencies / dependents of the package |
| `/` | filter by name |
| `f` | show only problems (failed, skipped, changed, never built) |
| `s` | sort: build order / name / status |
| `o` | open a shell in the package directory (`exit` to return) |
| `r` | reload |
| `?` | help |
| `q Esc` | quit |

---

## gitws — git repositories overview

```bash
gitws                # all repos under the current directory
gitws src/           # under a given path
gitws -d 3           # limit search depth (default 8)
gitws --no-fetch     # don't fetch at startup
gitws -l             # print the table and exit
```

### How it works

- **Finding repos:** walks the directory tree looking for `.git` (directories or files, so
  submodules/worktrees too). Skips hidden dirs and `build`, `install`, `log`, `node_modules`,
  `__pycache__`, `venv`. If nothing is found, it uses the repo containing the current directory.
- **Fetch:** at startup it runs `git fetch --all --prune` on all repos in the background, so
  ahead/behind counts are up to date. Background git commands never ask for passwords
  (`GIT_TERMINAL_PROMPT=0`, ssh `BatchMode=yes`): a repo that needs credentials shows `⚠fetch`.
  Interactive commands (pull, push, commit) run in the terminal normally and can prompt.
- **Status columns:**
  - sync: `↑N` commits to push, `↓N` commits to pull, `✓ sync`, `no upstream`, `detached`
  - changes: `!N` conflicts, `+N` staged, `~N` modified, `?N` untracked, or `clean`
  - ongoing operation: MERGE, REBASE, CHERRY, REVERT, BISECT; and stash count
- **Sort by urgency** (`s`): errors, conflicts, ongoing merge/rebase, behind, ahead, dirty, stash.

### Keys — repository list

| Key | Action |
|-----|--------|
| `ENTER / →` | open the repo |
| `↑↓ j k PgUp PgDn g G` | move |
| `f` / `F` | fetch selected / fetch all (background) |
| `p` / `P` | pull / push selected |
| `U` | `pull --ff-only` every repo that is behind and has no local changes |
| `l` | log |
| `d` | show only repos that need attention |
| `/` | filter by name |
| `s` | sort by name / urgency |
| `o` | open a shell in the repo |
| `r` | reload |
| `?` | help |
| `q Esc` | quit |

### Keys — inside a repo

| Key | Action |
|-----|--------|
| `↑↓ j k` | select file |
| `SPACE` | stage / unstage the file |
| `a` / `u` | stage all (`git add -A`) / unstage all (`git reset`) |
| `ENTER / →` | diff of the file |
| `D` | full diff (staged + unstaged) |
| `c` | commit (asks for the message) |
| `C` | commit with editor (`git commit -v`) |
| `p` / `P` | pull / push |
| `f` | fetch |
| `l` | graph log (HEAD and upstream) |
| `x` | discard changes to the file / delete untracked file (asks confirmation) |
| `z` / `Z` | `git stash push` / `git stash pop` |
| `o` | open a shell |
| `r` | reload |
| `q ← Esc` | back to the list |

---

## cbfail — build log viewer

```bash
cbfail          # interactive viewer of log/latest_build
cbfail -s       # plain summary
cbfail -b 2     # 2nd most recent build
cbfail -l log/build_2026-...   # specific build dir
```

Package list: `↑↓` move, `ENTER` open, `f` only problems, `b` change build, `q` quit.
Log view: `a` all lines, `e` errors, `w` warnings+errors, `n/N` next/prev problem, `/` search,
`g/G` top/bottom, `q/Esc` back.

---

## License

Apache License 2.0 — see [LICENSE](LICENSE). Release notes are in [CHANGELOG.md](CHANGELOG.md).
