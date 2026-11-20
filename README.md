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
├── haybale-comfyui/
└── marketplace/            ← the curated catalogue (no library code)
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
gh repo clone going-haywire/haybale-protokol
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
repository is a folder of its own, and *Studio (all libraries)* in *Run and Debug*
starts the studio from this environment. Give each repository folder its own
interpreter (its `.venv`) for its tests.

**Run the full studio from any checkout or worktree**: `bin/haywire-studio`, run inside
it (link it onto your `PATH` once: `ln -s "$PWD/bin/haywire-studio" ~/.local/bin/`).

```sh
cd <checkout or worktree>
haywire-studio            # arguments go to haywire, e.g. --no-browser
```

In a main checkout it is `uv run --project <this folder> haywire`. In a worktree, for
that run only, the worktree's `packages/*` and `barn/*` replace the umbrella's copies,
and the worktree gets a studio port of its own (the lowest free one above 8124, kept in
its `.haywire/settings.json`, by `haywire-repo/scripts/worktree_setup.py`), so several
worktrees run side by side. A worktree outside this folder gets a link beside it for
each `../<repository>` its sources name, pointing at this folder's copy. Code that lists
every installed distribution sees a replaced package twice; library discovery does not.

The worktree's sources must name every haywire package it depends on; one missing is
installed from PyPI for the run instead of this folder's copy.

### Herdr

Herdr's New worktree puts a worktree at `~/.herdr/worktrees/<repo>/<branch>` and opens
it as a space. With these, the studio starts from any space:

`~/.config/herdr-app/keys.json`, under `"global"` (`cmd-shift-s` opens a `studio` tab):

```json
"cmd-shift-s": {"command": "tab.new", "args": {"name": "studio", "argv": ["haywire-studio"]}}
```

`~/.config/herdr-app/programs.json` (*Haywire studio* in the agent menu):

```json
{ "programs": { "haywire-studio": { "name": "Haywire studio", "run": ["haywire-studio"], "match": ["haywire"] } } }
```

The command runs on the space's machine, so a remote Linux space needs this umbrella
there too (as the parent folder of its repositories) and `haywire-studio` on its login
`PATH`. Its studio listens on 127.0.0.1: `ssh -L <port>:localhost:<port> <machine>`.

## Adding a repository

Clone it into this folder and run `bin/umbrella-sync`. It lists every package it finds
(`haywire-repo/packages/*`, and `barn/*` of every repository here) in `dependencies` and
`[tool.uv.sources]` of `pyproject.toml` (relative path, `editable = true`), adds the
repository's folder to `haywire.code-workspace`, and runs `uv sync`. Run it again
whenever a repository gains or loses a library; `bin/umbrella-sync --check` only reports
drift. The repository's own `pyproject.toml` must use relative path sources too: uv
applies them here.

## Agent sessions

Start agent sessions inside a repository, never in this folder: the rules and hooks live
in each repository. This folder has no `CLAUDE.md` on purpose, because agents also load
`CLAUDE.md` files from parent folders.
