# Podman, Systemd, and Systemctl: Man Pages and Help Guide

This reference provides the most useful `--help` and `man` commands for diving deep into the usage of Podman, Systemd, and Systemctl utilities.

---

## Podman: Help and Man Pages

### General Help

```bash
podman --help
podman <subcommand> --help      # Example: podman run --help
```

### Man Pages

| Man Page Command | Description |
|------------------|-------------|
| `man podman` | Main Podman CLI reference |
| `man podman-run` | Details of the `run` subcommand |
| `man podman-container` | Managing container lifecycle |
| `man podman-volume` | All volume operations |
| `man podman-network` | Networking with Podman |
| `man podman-generate` | Generate systemd or Kubernetes files |

### Search for Related Topics

```bash
apropos podman
```

---

## Systemctl (Systemd CLI Interface)

### General Help

```bash
systemctl --help
```

### Man Pages

| Man Page Command | Description |
|------------------|-------------|
| `man systemctl` | Main CLI tool to control systemd |
| `man systemd.unit` | Overview of all unit file types |
| `man systemd.service` | Service unit configuration |
| `man systemd.timer` | Timer unit configuration |
| `man systemd.target` | Target grouping configuration |
| `man systemd.socket` | Socket activation settings |
| `man systemd.exec` | Exec behavior in unit files |
| `man systemd.directives` | Full list of supported directives |

---

## Systemd Core Utilities

| Man Page Command | Description |
|------------------|-------------|
| `man systemd` | Overview of the init system |
| `man journalctl` | Query and filter systemd logs |
| `man loginctl` | Manage user sessions (e.g., linger) |
| `man systemd.resource-control` | Limit CPU and memory usage |
| `man systemd.special` | Predefined special targets |

---

## Tips for Using `man` Effectively

- Search for related topics:
  ```bash
  man -k podman
  man -k systemd | grep timer
  ```

- Search inside man pages:
  ```
  /Restart=
  ```

- Controls:
  - `q` to quit
  - `space` to scroll forward
  - `b` to scroll backward

---

Keep this handy as your quick-access reference for in-depth help and command discovery.
