# systemd vs Podman: Service Management Reference

This table outlines the best practices for managing Podman containers using `systemctl` and shows which system-level actions correspond to Podman actions (if any). It also includes where service metadata is stored, where logs go, and what command to use for inspection and control.

---

## Systemd vs Podman Container Management

| Task                          | `systemctl` Command (User or Root)         | Podman Equivalent          | File/Directory Involved                                  | Notes |
|-------------------------------|---------------------------------------------|-----------------------------|-----------------------------------------------------------|-------|
| **Start container**          | `systemctl start <unit>`                   | `podman start <container>` | `/etc/systemd/system/` or `~/.config/systemd/user/`       | Controlled by unit file (`.service`) |
| **Stop container**           | `systemctl stop <unit>`                    | `podman stop <container>`  | Same as above                                             | `ExecStop` defined in unit |
| **Restart container**        | `systemctl restart <unit>`                | `podman restart`           | Same as above                                             | Calls `stop` then `start` |
| **Enable on boot/login**     | `systemctl enable <unit>`                 | *n/a*                      | Symlink in `/etc/systemd/system/multi-user.target.wants/` | Boot-persistence |
| **Disable on boot**          | `systemctl disable <unit>`                | *n/a*                      | Removes symlink                                           | |
| **View service logs**        | `journalctl -u <unit>`                    | `podman logs`              | `/var/log/journal/` or `/run/log/journal/`               | systemd-managed |
| **Monitor in real time**     | `journalctl -fu <unit>`                   | `podman logs -f`           | Same as above                                             | |
| **View service status**      | `systemctl status <unit>`                 | `podman inspect`           | Uses cgroups and unit metadata                            | |
| **Check if running**         | `systemctl is-active <unit>`              | `podman ps`                | *n/a*                                                     | |
| **Check if enabled**         | `systemctl is-enabled <unit>`             | *n/a*                      | *n/a*                                                     | |
| **Reload unit files**        | `systemctl daemon-reexec && systemctl daemon-reload` | *n/a* | *n/a* | Required after editing `.service` files |
| **Edit service inline**      | `systemctl edit --user <unit>`            | *n/a*                      | Opens drop-in override in `~/.config/systemd/user/<unit>.d/` | |
| **List all units**           | `systemctl list-units --type=service`     | `podman ps -a`             | *n/a*                                                     | |
| **Create unit file**         | `podman generate systemd --name <ctr>`    | *generates config*         | Outputs `.service` file to `.`                            | |
| **Enable lingering (user)**  | `loginctl enable-linger $USER`            | *n/a*                      | Enables user services without active login                | |

---

## Common File Paths

| Path | Purpose |
|------|---------|
| `/etc/systemd/system/` | System-wide service units |
| `~/.config/systemd/user/` | User-scoped service units |
| `/run/systemd/system/` | Runtime-created units |
| `/var/log/journal/` or `/run/log/journal/` | Persistent and volatile journald logs |
| `/etc/containers/containers.conf` | Podman container settings |
| `~/.config/containers/containers.conf` | User-level container settings |
| `/etc/cni/net.d/` or `~/.config/cni/net.d/` | CNI network configurations |

---

## Best Practices Summary

- Use `systemctl` for all **long-running**, **auto-restarting**, and **boot-enabled** Podman containers.
- Use `podman` CLI for **manual** runs, **debugging**, **dev workflows**, and **adhoc commands**.
- Always run `systemctl daemon-reexec && daemon-reload` after modifying unit files.
- Use `journalctl` instead of `podman logs` for system-managed logging (it’s safer and more complete).
- To monitor crash recovery: use `Restart=on-failure` in your `.service` file and check restart count with `systemctl show <unit> -p NRestarts`.

---

## Sample Check and Trace Workflow

```bash
systemctl status my-container.service         # Check if it's running
journalctl -u my-container.service            # View logs
systemctl show my-container.service           # Show properties
systemctl is-enabled my-container.service     # See if enabled at boot
```

---

## Recommendation

When managing Podman containers with systemd:
- Think of `systemctl` as your **orchestrator**
- Think of `podman` as your **runtime tool**
