# Changelog

All notable changes to this project are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project follows [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.0.0] - 2026-11-11

### Added
- `cbws`: interactive colcon build manager for ROS 2 workspaces.
  - Package list with last build status, build type, age, errors/warnings, "changed" flag.
  - Build only / up to / from selected packages, whole workspace, failed or changed packages.
  - Clean and clean rebuild, per-workspace build options, build type cycling.
  - Dependency view, filters, sorting, shell in package directory.
  - Detection of external `colcon build` runs; `-s` plain status table.
- `gitws`: overview of all git repositories under a directory.
  - Background fetch with no password prompts; ahead/behind, changes, stash, merge/rebase state.
  - Stage/unstage, diff, commit, pull, push, ff-only pull of all repos, stash, discard.
  - Sort by urgency, attention filter; `-l` plain table.
- `cbfail`: colcon build log viewer, used by `cbws` and usable on its own.
