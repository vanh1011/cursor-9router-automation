# Cursor 9Router Automation

macOS scripts that turn 9Router on or off, create a public tunnel, and update the new URL in Cursor automatically.

## Why

If you do not have a server to host 9Router permanently, you need to run 9Router on your local machine. However, Cursor does not allow a local HTTP URL to be used directly as the OpenAI Base URL, so a public tunnel is required.

Each time 9Router starts, the tunnel may generate a new URL. Copying that URL and manually updating Cursor every day is slow and error-prone.

## Solves This

These scripts automate the whole flow:

- Turn 9Router on or off with one command.
- Create a public tunnel and read the latest URL.
- Append `/v1` to the endpoint.
- Automatically update `openAIBaseUrl` in Cursor.
- Restart Cursor so the new configuration is applied.

## How It Works

### `router-cursor-on`

1. Checks required dependencies and Cursor's configuration database.
2. Starts 9Router at `127.0.0.1:20128`.
3. Enables the Quick Tunnel managed by 9Router and reads the public URL.
4. Closes Cursor safely and backs up the configuration database.
5. Updates `openAIBaseUrl` with the tunnel URL plus the `/v1` suffix.
6. Opens Cursor again after the update succeeds.

If the database update fails, the script restores the backup together with the related SQLite WAL/SHM files.

### `router-cursor-off`

1. Finds 9Router, the listener on port `20128`, and the related tunnel process.
2. Sends `SIGTERM` so the processes can stop safely.
3. Uses `SIGKILL` only if a process does not respond after the timeout.

## Requirements

- macOS.
- Cursor installed and launched at least once.
- 9Router CLI available in `PATH`.
- System utilities: `zsh`, `python3`, `pgrep`, `lsof`, `ps`, `sort`, `tr`, `osascript`, and `open`.
- 9Router local tunnel data available under `~/.9router`.

Check 9Router:

```zsh
command -v 9router
```

## Quick Start

Grant execute permission the first time:

```zsh
chmod +x router-cursor-on router-cursor-off
```

Start 9Router, enable the tunnel, and configure Cursor:

```zsh
./router-cursor-on
```

Stop 9Router and the tunnel:

```zsh
./router-cursor-off
```

> `router-cursor-on` may close and reopen Cursor. Save any work in progress before running it.

## Related Files and Data

- 9Router log: `~/.9router.log`
- Tunnel state: `~/.9router/tunnel/state.json`
- Cursor database:
  `~/Library/Application Support/Cursor/User/globalStorage/state.vscdb`
- Latest backup: `~/.cursor-state.vscdb.backup`

## Limitations

- These scripts are currently macOS-only.
- Port `20128` is hard-coded in both scripts.
- Cursor's internal settings storage may change after an update. Recheck the script if Cursor no longer picks up the Base URL.
