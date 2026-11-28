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
├── haybale-agents/
├── haybale-comfyui/
└── marketplace/            ← the curated catalogue (no library code)
```

Clone only the repositories this machine needs; `haywire-repo` is the one every machine
has.

`pyproject.toml`, `uv.lock` and `haywire.code-workspace` are not tracked. `bin/umbrella-sync`
writes them for each machine from the repositories cloned there, so two machines with
different libraries never fight over them. Each machine therefore resolves its own
third-party versions; the repositories' own tracked locks are what CI tests against.

## Set up on a new machine

```sh
gh repo clone going-haywire/umbrella haywire
cd haywire
gh repo clone going-haywire/haywire haywire-repo
# then any of these this machine needs:
gh repo clone going-haywire/haybale-spandrel
gh repo clone going-haywire/haybale-visiongraph
gh repo clone going-haywire/haybale-stream
gh repo clone going-haywire/haybale-comfyui
gh repo clone going-haywire/haybale-protokol
gh repo clone going-haywire/haybale-agents
gh repo clone going-haywire/marketplace
bin/umbrella-sync
```

## Use

```sh
bin/umbrella-sync                         # in this folder: list every package, then uv sync
cd haywire-repo && uv run --project .. haywire   # run the studio with every library
```

Before syncing, pull every repository, `haywire-repo` first: the sources of every
repository point into it, so a stale checkout fails the sync on a library it does not
have yet (such as `barn/haybale-audio`).

The studio uses the folder it starts in as its project (`.haywire/` settings, graphs,
port), so start it inside `haywire-repo` (or another project folder) with `--project ..`
pointing here, never in this folder itself.

In VS Code, open `haywire.code-workspace` (*File → Open Workspace from File…*): every
repository with libraries is a folder of its own (add others such as `marketplace` by
hand; the file is this machine's), and *Studio (all libraries)* in *Run and Debug*
starts the studio from this environment. Give each repository folder its own
interpreter (its `.venv`) for its tests.

**Run the full studio from any checkout or worktree**: `bin/haywire-studio`, run inside
it (link it onto your `PATH` once: `ln -s "$PWD/bin/haywire-studio" ~/.local/bin/`).

```sh
cd <checkout or worktree>
haywire-studio            # arguments go to haywire, e.g. --no-browser
```

It first runs `bin/umbrella-sync --no-sync --quiet`, so a library a repository gained
is in the run without a manual sync.
In a main checkout it is then `uv run --project <this folder> haywire`. In a worktree, for
that run only, the worktree's `packages/*` and `barn/*` replace the umbrella's copies,
and the worktree gets a studio port of its own (the lowest free one above 8124, kept in
its `.haywire/settings.json`, by `haywire-repo/scripts/worktree_setup.py`), so several
worktrees run side by side. A worktree outside this folder gets a link beside it for
each `../<repository>` its sources name, pointing at this folder's copy. Code that lists
every installed distribution sees a replaced package twice; library discovery does not.

The worktree's sources must name every haywire package it depends on; one missing is
installed from PyPI for the run instead of this folder's copy.

### Orchestrators

How to start the studio from an orchestrator (herdr, T3 Code, …) is on its page in core:
[Orchestrators](https://github.com/going-haywire/haywire/blob/master/docs/collaboration/orchestrators/index.md).

## Adding a repository

Clone it into this folder and run `bin/umbrella-sync`. It lists every package it finds
(`haywire-repo/packages/*`, and `barn/*` of every repository here) in `dependencies` and
`[tool.uv.sources]` of `pyproject.toml` (relative path, `editable = true`), adds the
repository's folder to `haywire.code-workspace`, and runs `uv sync`. A missing file is
written from the script's template first. Worktrees beside the repositories are skipped. Run it again
whenever a repository gains or loses a library; `bin/umbrella-sync --check` only reports
drift. The repository's own `pyproject.toml` must use relative path sources too: uv
applies them here.

## Agent sessions

The umbrella runs core's session hooks: `.claude/settings.json` points at
`haywire-repo/dev/hooks/`, so a lead started in this folder gets its role, the T3/herdr
slot (it is a main folder, so a lead) and the git guards, like one started in a repository.
The hooks resolve only where `haywire-repo` sits inside the project folder, that is in this
main folder. A worktree of this repository has no `haywire-repo`, so there each hook fails
to start (Claude Code reports it and goes on): run umbrella sessions here. Task work still
runs inside a repository, whose own rules and hooks apply there. This folder has
no `CLAUDE.md` on purpose, because agents also load `CLAUDE.md` files from parent folders.
