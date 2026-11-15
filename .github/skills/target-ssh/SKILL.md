---
name: target-ssh
description: 'Connect to the target board over SSH (root@radxa-zero-3e-custom.lan) and run commands on it. USE FOR: logging into the board, running remote commands, checking files, deploying or testing firmware, kernel modules on the device. DO NOT USE FOR: local host commands or building on the host.'
---

# Target SSH

## Connection

Use this exact command to open an interactive shell:

```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" root@radxa-zero-3e-custom.lan
```

Host key checking is disabled and no known_hosts file is written, so the board's key changes after reflashing do not block the connection.

## Running Remote Commands

Append the command in quotes to run it non-interactively and return:

```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" -o "BatchMode=yes" root@radxa-zero-3e-custom.lan "<command>"
```

- `BatchMode=yes` makes ssh fail instead of prompting for a password. Omit it if a password prompt is expected.
- Run one ssh command per terminal call. Wait for the output before running the next.
- Use `run_in_terminal` in sync mode for one-shot commands.

## Copying Files

```bash
scp -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" <local-path> root@radxa-zero-3e-custom.lan:<remote-path>
```

## Notes

- If the board is unreachable, check that it is powered and on the network, and that `radxa-zero-3e-custom.lan` resolves (mDNS). Do not change the host key options to work around it.
- If a password is requested, ask the user to type it into the terminal. Never pass passwords through chat or tool arguments.
- Do not write credentials into this file.
