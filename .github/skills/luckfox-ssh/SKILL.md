---
name: luckfox-ssh
description: 'Connect to the Luckfox Lyra Plus board over SSH (root@luckfox-lyra-plus.lan) and run commands on it. USE FOR: logging into the board, running remote commands, checking files, deploying or testing firmware, kernel modules, or rk3506_rproc on the device. DO NOT USE FOR: local host commands or building on the host.'
---

# Luckfox Lyra Plus SSH

## Connection

Use this exact command to open an interactive shell:

```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" root@luckfox-lyra-plus.lan
```

Host key checking is disabled and no known_hosts file is written, so the board's key changes after reflashing do not block the connection.

## Running Remote Commands

Append the command in quotes to run it non-interactively and return:

```bash
ssh -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" -o "BatchMode=yes" root@luckfox-lyra-plus.lan "<command>"
```

- `BatchMode=yes` makes ssh fail instead of prompting for a password. Omit it if a password prompt is expected.
- Run one ssh command per terminal call. Wait for the output before running the next.
- Use `run_in_terminal` in sync mode for one-shot commands.

## Copying Files

```bash
scp -o "UserKnownHostsFile=/dev/null" -o "StrictHostKeyChecking no" <local-path> root@luckfox-lyra-plus.lan:<remote-path>
```

## Notes

- If the board is unreachable, check that it is powered and on the network, and that `luckfox-lyra-plus.lan` resolves (mDNS). Do not change the host key options to work around it.
- If a password is requested, ask the user to type it into the terminal. Never pass passwords through chat or tool arguments.
- Do not write credentials into this file.
