# tandem

Launch a tmux session with SSH and an AI assistant (Claude Code) side-by-side.
The agent can read your terminal and send commands to the remote host using
standard tmux commands.

```
┌─────────────────────────────┬──────────────────────────────┐
│  SSH: host.my.domain.com    │  Claude Code                 │
│  jcooper@host:~$            │                              │
│                             │  > read the ssh pane and     │
│  (you type here)            │    help me debug this issue  │
└─────────────────────────────┴──────────────────────────────┘
```

## Install

```bash
# Symlink into your PATH
ln -s /path/to/tandem/tandem ~/.local/bin/tandem
```

## Dependencies

| Dependency | Purpose | Install |
|---|---|---|
| `tmux` | Terminal multiplexer | `sudo apt install tmux` or `sudo dnf install tmux` |
| `claude` | AI assistant ([Claude Code](https://claude.com/product/claude-code)) | `curl -fsSL https://claude.ai/install.sh \| bash` |
| `python3` | Workspace trust setup | `sudo apt install python3` or `sudo dnf install python3` |
| `ssh` | Remote connection | `sudo apt install ssh` or `sudo dnf install ssh` |

## Usage

```bash
tandem <hostname>                # SSH using your .ssh/config
tandem host                      # bare name auto-expands to host.my.domain.tld
tandem jcooper@10.0.1.50         # explicit user
tandem -l vertical host         # stacked layout (SSH on top, agent on bottom)
```

The SSH pane is focused on startup. Switch panes with `Ctrl-B` then arrow keys.

If a tmux session for that host already exists, `tandem` reattaches to it.

## How it works

1. Creates a per-host workspace at `~/.tandem/<hostname>/`
2. Starts a tmux session with two panes: SSH (left) and Claude Code (right)
3. A `CLAUDE.md` in the workspace teaches the agent how to read and interact
   with the SSH pane using `tmux capture-pane` and `tmux send-keys`

No MCP servers or extra tooling required — the agent uses its built-in Bash
tool to run tmux commands directly.

## Configuration

Copy the example config to `~/.tandem/tandem.conf`:

```bash
mkdir -p ~/.tandem
cp /path/to/tandem/tandem.conf.example ~/.tandem/tandem.conf
```

| Variable | Default | Description |
|---|---|---|
| `TANDEM_AGENT` | `claude` | AI agent command |
| `TANDEM_LAYOUT` | `horizontal` | `horizontal` (side-by-side) or `vertical` (stacked) |
| `TANDEM_SSH_CMD` | `ssh` | SSH command (e.g., `mosh`) |
| `TANDEM_BASE_DIR` | `~/.tandem` | Base directory for per-host workspaces |
| `TANDEM_DEFAULT_DOMAIN` | `my.domain.tld` | Domain appended to bare hostnames that don't resolve |

## Per-host workspace

Each host gets a workspace at `~/.tandem/<hostname>/` containing a `CLAUDE.md`
with instructions for the agent (tmux pane targets, usage examples, guidelines).

The file is created on first run and never overwritten, so you can customize it
per host — add debugging checklists, host-specific context, etc.

## Tmux cheat sheet

| Keys | Action |
|---|---|
| `Ctrl-B` `←`/`→` | Switch between SSH and agent panes |
| `Ctrl-B` `d` | Detach (session keeps running; reattach with `tandem <host>`) |
| `Ctrl-B` `z` | Zoom current pane to full screen (toggle) |
| `Ctrl-B` `[` | Scroll mode (navigate with arrows, `q` to exit) |
