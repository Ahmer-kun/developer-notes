# Error: Port Already in Use (EADDRINUSE)

## Symptoms

```
Error: listen EADDRINUSE: address already in use :::3000
```

The server refuses to start.

## Likely causes

- A previous run of the same app is still running (terminal closed badly, watcher didn't stop, another terminal tab).
- A different program uses that port.
- On macOS, system services can occupy some ports. AirPlay Receiver uses port 5000 (and 7000) on recent macOS versions, which surprises people running dev servers on 5000.
- Docker container publishing the same host port.

## Checks

Find what's listening.

```bash
# macOS and Linux
lsof -i :3000

# Linux
ss -tulpn | grep 3000

# Windows (cmd)
netstat -ano | findstr :3000
```

The output includes a PID (process ID). `ss` may need `sudo` to show process names for other users.

Docker:

```bash
docker ps
```

Look at the `PORTS` column.

## Fix

Stop the process using it.

```bash
kill <pid>          # try this first
kill -9 <pid>       # only if it ignores the normal signal
```

Windows:

```
taskkill /PID <pid> /F
```

Or run your app on another port. Make the port configurable:

```js
const port = process.env.PORT || 3000;
app.listen(port, () => console.log(`listening on ${port}`));
```

```bash
PORT=3001 node app.js
```

## Why it happened

Only one process can listen on a given address and port at a time. The OS rejects the second bind. Dev tools that restart on file change can leave orphaned children if the parent is killed abruptly.

## Prevention

- Stop dev servers with Ctrl+C rather than closing the terminal window.
- Use `PORT` from the environment; don't hard-code.
- Use graceful shutdown handlers for `SIGINT`/`SIGTERM` so the server closes its listener.
- Keep a list of ports your projects use.

## Notes

- Not the same as `ECONNREFUSED`, which is the opposite (nothing listening where you tried to connect).
- `EACCES` on `listen` for ports below 1024 on Linux/macOS means privileges are needed, a different problem.

## Related

- [Linux cheatsheet](../../devops/linux/linux-cheatsheet.md)
- [TCP vs UDP](../../computer-science/networking/tcp-vs-udp.md)
- [Environment variables not loading](env-variables-not-loading.md)
