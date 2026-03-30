# apt-tmux

A tmux session manager with interactive picking, collision handling, and shebang-based session scripts.

## Installation

Put `apt-tmux` somewhere on your PATH:

```bash
ln -s /path/to/apt-tmux/apt-tmux ~/.local/bin/apt-tmux
```

## Commands

### Auto (no args)

```bash
apt-tmux
```

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
