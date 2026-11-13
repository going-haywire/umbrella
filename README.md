# haywire umbrella

One environment with every haywire package from every repository in this folder,
installed editable, for running the full studio with all libraries. Each repository
keeps its own environment for its own tests and CI; this one never changes them.

## Layout

The repositories are cloned side by side in this folder:

```
haywire/                    ← this repository (umbrella files only)
├── haywire-repo/
├── haybale-spandrel/
├── haybale-visiongraph/
├── haybale-stream/
├── haybale-protokol/
└── haybale-comfyui/
```

All paths in `pyproject.toml` are relative, so the same files work on every machine with
this layout.

## Set up on a new machine

```sh
gh repo clone going-haywire/umbrella haywire
cd haywire
gh repo clone going-haywire/haywire haywire-repo
gh repo clone going-haywire/haybale-spandrel
gh repo clone going-haywire/haybale-visiongraph
gh repo clone going-haywire/haybale-stream
gh repo clone going-haywire/haybale-comfyui
uv sync
```

`haybale-protokol` has no repository on GitHub yet. Until it has, a machine without a copy
of it removes its two lines from `pyproject.toml` (in `dependencies` and
`[tool.uv.sources]`) before `uv sync`, and does not commit that change.

## Use

```sh
uv sync                           # build or update the umbrella environment
uv run haywire                    # run the studio with every library
```

In VS Code, open `haywire.code-workspace` (*File → Open Workspace from File…*): every
repository is a folder of its own, and *Studio (all libraries)* in *Run and Debug*
starts the studio from this environment. Give each repository folder its own
interpreter (its `.venv`) for its tests.

**Run the full studio against a worktree's code**: overlay the worktree's packages for
one run; the umbrella itself does not change.

```sh
uv run --with-editable <worktree>/packages/haywire-core \
       --with-editable <worktree>/barn/haybale-core \
       haywire
```

Overlay each package the worktree changes. Code that lists every installed distribution
then sees an overlaid package twice; library discovery does not.

## Adding a repository

Clone it into this folder, then add each of its `barn/*` libraries to `dependencies` and
`[tool.uv.sources]` in `pyproject.toml` (relative path, `editable = true`), its folder to
`haywire.code-workspace`, and run `uv sync`. Its own `pyproject.toml` must use relative
path sources too: uv applies them here.

## Agent sessions

Start agent sessions inside a repository, never in this folder: the rules and hooks live
in each repository. This folder has no `CLAUDE.md` on purpose, because agents also load
`CLAUDE.md` files from parent folders.
