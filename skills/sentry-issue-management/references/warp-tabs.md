# Warp tabs for the new worktrees

One Warp tab per worktree, each already SSH'd into the dev box, sitting in that worktree, with the
agent running. Warp reads the files on the owner's Mac, so this side only writes them and hands
over one command.

Warp's **launch configurations are legacy** — a single YAML of windows and tabs. Newer Warp
ignores both the `warp://launch/...` URI and the `Launch Configuration` palette entry, which is
what "nothing happened" looked like on 2026-08-28. **Tab configs replace them**: one TOML file per
tab, in `~/.warp/tab_configs/`. Write tab configs.

## 1. Write one TOML per worktree on the dev box

```bash
mkdir -p /tmp/warp-tab-configs
```

Then one file per worktree, named for it — `/tmp/warp-tab-configs/<worktree>.toml`:

```toml
name = "class-roster-null-guard"
title = "class-roster-null-guard"
color = "magenta"

[[panes]]
id = "main"
type = "terminal"
is_focused = true
commands = ["ssh -t willow 'cd /root/code/class-roster-null-guard && claude \"Use the investigate-sentry-issues skill on 6789, 6801\"'"]
```

Five things that break it if changed:

- **The tab opens the agent on its own investigation.** The trailing argument is the agent's first
  prompt: it names the `investigate-sentry-issues` skill and that row's Sentry ids, comma-separated,
  and nothing else — the ids belong to that worktree alone. Name the skill in words rather than as a
  host slash command — the same prompt then works on any agent that carries the skill, and this file
  is host-neutral. Escape the inner double quotes as `\"`;
  plain `claude` with no argument lands the owner in an empty session with the ids on a table he
  has to scroll back to.
- **No `directory` key.** The pane starts on the Mac, and any local path that does not exist stops
  the tab before the command runs. Omitting it means "wherever Warp opens".
- `ssh -t` — without a TTY the agent starts and immediately exits.
- Single quotes around the remote command, so the Mac's shell does not expand it locally.
- `color` is `magenta` on every tab (owner, 2026-08-28) — his theme makes it purple. It is not a
  priority signal: the table above carries priority, and colour-coding six tabs by P-level only
  made them look arbitrary. Valid values are `black`, `red`, `green`, `yellow`, `blue`, `magenta`,
  `cyan`, `white`, lowercase, and the actual shade comes from the owner's Warp theme.

`willow` is the owner's SSH alias for the dev box. If a different alias is in use, ask once rather
than guessing.

## 2. The agent must be on the default PATH

`ssh host 'cmd'` is neither interactive nor a login shell, so it reads neither `.bashrc` nor
`.bash_profile`. On this box `nvm` — and therefore `claude` — is set up in `.bashrc` alone, so the
tab died with `claude: command not found` until `claude` was symlinked somewhere already on the
default PATH:

```bash
ln -sfn /root/.nvm/versions/node/<version>/bin/claude /usr/local/bin/claude
ln -sfn /root/.nvm/versions/node/<version>/bin/node /usr/local/bin/node
```

Confirm before handing over, and re-point the symlinks after a Node upgrade:

```bash
env -u PATH bash -c 'command -v claude'
```

`bash -lc` is NOT the fix. A login shell reads `.bash_profile`/`.profile`, not `.bashrc`, so it
fails identically and only looks like it works from a shell that already had the PATH.

## 3. Hand the owner one command

This is the whole handoff. It copies every file to the Mac and opens every tab:

```bash
mkdir -p ~/.warp/tab_configs && \
scp "willow:/tmp/warp-tab-configs/*.toml" ~/.warp/tab_configs/ && \
for t in <worktree-1> <worktree-2> <worktree-3>; do open "warp://tab_config/$t"; done
```

Fill the loop with the table's worktree names, in priority order. **Quote the scp source.** Warp
opens tabs in that order, and an unquoted `*` makes zsh try to expand it on the Mac and fail with
`zsh: no matches found`.

Never run it from here — it only works on the machine Warp is installed on, and opening the
owner's terminal windows is their call.

## If the tabs do not open

The command palette (`⌘P`) does **not** list tab configs, and neither does the legacy
`⌃⌘L` launch-configuration palette — searching either finds nothing even when the files are
correctly installed. The fallback is the **+** button in the tab bar, which lists every tab config.
