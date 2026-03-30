# apt-tmux

A tmux session manager with interactive picking, collision handling, and shebang-based session scripts.

## Installation

Put `apt-tmux` somewhere on your PATH:

```bash
ln -s /path/to/apt-tmux/apt-tmux ~/.local/bin/apt-tmux
```

## Commands

### Default (no args)

```bash
apt-tmux
```

Opens the interactive session manager. Equivalent to `apt-tmux manage`.

### auto

```bash
apt-tmux auto
```

Smart attach without the manager UI:

- No sessions → creates a new one and attaches
- One session → attaches to it
- Multiple sessions → interactive picker (with option to create new)

### new

```bash
apt-tmux new [name] [-- cmd...]
```

Creates a new session. If the name already exists, prompts:

```
Session 'work' already exists.
  a) Attach
  r) Replace (kill and recreate)
  k) Kill (and exit)
  n) New (choose a different name)
  q) Cancel
```

With a command:

```bash
apt-tmux new build -- make -j8
```

### attach

```bash
apt-tmux attach [name]
```

Attaches to the named session. If no name is given and multiple sessions exist, shows a picker. Errors if the session does not exist (never creates one).

### kill

```bash
apt-tmux kill [name]
```

Kills the named session. If no name is given and multiple sessions exist, shows a picker.

### rename

```bash
apt-tmux rename [old] [new]
```

Renames a session. If `old` is omitted and multiple sessions exist, shows a picker. If `new` is omitted, prompts for it.

### replace

```bash
apt-tmux replace [name] [-- cmd...]
```

Kills the existing session (if any) and creates a fresh one. No prompts.

### list

```bash
apt-tmux list
```

Lists session names, one per line. Suitable for scripting.

### manage

```bash
apt-tmux manage
```

Interactive session manager. Shows a session picker; selecting a session opens an action menu:

```
  a) Attach
  n) Rename
  r) Replace (kill and recreate)
  k) Kill
  b) Back
  q) Cancel
```

After each action (except attach and replace, which attach to a session), returns to the session list. Use `r) Refresh` in the session picker to reload the list.

## --no-pick flag

Commands that would normally show an interactive picker (`attach`, `kill`, `rename`, and no-args auto mode) accept `--no-pick`. When set, they fail with an error instead of showing the picker — useful for scripting.

```bash
apt-tmux attach --no-pick work   # attach to 'work', or error if it doesn't exist
apt-tmux kill --no-pick          # error if multiple sessions (can't pick non-interactively)
apt-tmux --no-pick               # error if multiple sessions exist
```

## Shebang session scripts

Create an executable file with `#!/usr/bin/env apt-tmux` as the interpreter. Running the script manages a named tmux session.

```bash
#!/usr/bin/env apt-tmux
#apt:session=claude
#apt:on-exists=attach
#apt:on-missing=create

claude
```

Make it executable and run it:

```bash
chmod +x ~/bin/claude
~/bin/claude
```

- If a session named `claude` exists → attaches
- If not → creates one running `claude` and attaches

### Metadata directives

| Directive | Values | Default |
|-----------|--------|---------|
| `#apt:session=` | session name | script filename (no extension) |
| `#apt:on-exists=` | `attach` \| `new` \| `replace` \| `kill` \| `ask` | `ask` |
| `#apt:on-missing=` | `create` \| `error` \| `ask` | `create` |

The first non-comment, non-blank line after the directives is the command to run when creating a new session.

### Values

**on-exists:**
- `attach` — attach to the existing session
- `new` — prompt for a new name and create a fresh session
- `replace` — kill the existing session and create a fresh one
- `kill` — kill the session and exit
- `ask` — prompt (attach / replace / kill / new / cancel)

**on-missing:**
- `create` — create the session and attach
- `error` — exit with an error
- `ask` — prompt before creating

## Requirements

- bash 4+
- tmux
