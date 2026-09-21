# Linux CLI Cheatsheet

Commands that come up constantly. Flags shown are common GNU/Linux ones; macOS (BSD) versions differ in places, noted where it matters.

## Files and directories

| Command | What it does | Example |
|---|---|---|
| `pwd` | print current directory | |
| `ls -la` | list all files with details | `ls -la ~/projects` |
| `cd -` | go back to previous directory | |
| `mkdir -p` | create nested directories, no error if they exist | `mkdir -p a/b/c` |
| `cp -r` | copy directories | `cp -r src backup` |
| `mv` | move or rename | `mv old.txt new.txt` |
| `rm -r` | remove directory tree (no trash, no undo) | `rm -r build/` |
| `find` | search by name/type/etc. | `find . -name "*.log" -mtime +7` |
| `du -sh` | size of a directory | `du -sh node_modules` |

Check what a wildcard expands to (`echo *.log`) before pairing it with `rm`.

## Reading files

| Command | What it does | Example |
|---|---|---|
| `cat` | print a file | `cat package.json` |
| `less` | scroll through a file (`q` to quit, `/` to search) | `less app.log` |
| `head -n` / `tail -n` | first / last lines | `tail -n 50 app.log` |
| `tail -f` | follow a file as it grows | `tail -f app.log` |
| `wc -l` | count lines | `wc -l data.csv` |

## Searching text

```bash
grep -rn "TODO" src/
```

`-r` recurse, `-n` show line numbers. Other useful flags: `-i` ignore case, `-v` invert match, `-l` only file names, `-C 3` context lines.

## Pipes and redirection

A pipe sends one command's output to another's input.

```bash
ps aux | grep node
sort names.txt | uniq -c | sort -rn | head
```

The second line counts duplicate lines and shows the most frequent first. (`uniq` only collapses *adjacent* duplicates, hence the first `sort`.)

| Syntax | Meaning |
|---|---|
| `>` | write stdout to file (overwrites) |
| `>>` | append stdout to file |
| `2>` | redirect stderr |
| `2>&1` | send stderr to the same place as stdout |
| `<` | read stdin from file |

`command > out.txt 2>&1` captures both streams in one file.

## Permissions

`ls -l` shows something like `-rwxr-xr-- 1 user group ...`.

- First character: file type (`-` file, `d` directory).
- Then three sets: owner, group, others, each `rwx` (read, write, execute).
- Numeric: r=4, w=2, x=1. So `rwx` = 7, `r-x` = 5, `r--` = 4.

| Command | What it does |
|---|---|
| `chmod 755 script.sh` | owner rwx, group and others r-x |
| `chmod +x script.sh` | add execute for everyone (subject to umask rules for `+`) |
| `chown user:group file` | change owner/group (usually needs sudo) |

For directories, `x` means "can enter / access items inside".

A "Permission denied" error: check `ls -l` on the file *and* its parent directories.

## Processes and ports

| Command | What it does | Example |
|---|---|---|
| `ps aux` | list processes | `ps aux \| grep node` |
| `top` / `htop` | live resource usage | |
| `kill <pid>` | ask a process to terminate (SIGTERM) | `kill 1234` |
| `kill -9 <pid>` | force kill (SIGKILL), no cleanup | last resort |
| `lsof -i :3000` | what's using port 3000 | works on Linux and macOS |
| `ss -tulpn` | listening TCP/UDP sockets with process (Linux) | may need `sudo` to see other users' processes |

See [port already in use](../../debugging/node/port-already-in-use.md).

## Environment variables

```bash
echo $PATH               # print a variable
export API_URL=http://localhost:5000   # set for this shell and child processes
API_URL=http://localhost:5000 node app.js   # set for one command only
env                      # list all
```

Variables set with `export` last only for that shell session unless added to a shell config file (`~/.bashrc`, `~/.zshrc`, etc.). See [env variables not loading](../../debugging/node/env-variables-not-loading.md).

## Networking

```bash
curl -i https://example.com                 # response headers + body
curl -X POST https://api.example.com/items \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}'                      # JSON POST
curl -s -o /dev/null -w "%{http_code}\n" URL   # only the status code
ping example.com
dig +short example.com                       # see DNS note
```

`curl -v` also shows the request and TLS details. Bypasses browser-only rules like [CORS](../../backend/apis/cors.md), which is useful for narrowing down a problem.

## Package management (Debian/Ubuntu)

```bash
sudo apt update          # refresh package lists
sudo apt install curl    # install
apt search keyword
```

Other distros use different tools (`dnf`, `pacman`, `apk`); macOS commonly uses Homebrew (`brew install`).

## Remember

- Everything is a file; permissions are per owner/group/others.
- Pipes chain small tools.
- `rm` and `>` don't ask twice.
