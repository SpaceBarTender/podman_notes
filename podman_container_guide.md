# Podman Container Guide

## 1. What is Podman? How Does it Differ from Docker?

Podman is a container engine that provides a daemonless and rootless alternative to Docker. It uses the OCI (Open Container Initiative) format and integrates natively with systemd.

### Key Differences

| Feature           | Docker               | Podman                |
|-------------------|----------------------|------------------------|
| Daemon            | Yes (`dockerd`)      | No daemon (per-process) |
| Rootless          | Not by default       | Default                |
| Systemd Integration | Manual              | Native (`generate systemd`) |
| Compose Support   | Native               | Indirect (`podman-compose`) |

---

## 2. Container Lifecycle with Podman

```bash
podman build -t my-app .               # Build image from Dockerfile
podman run -d --name app my-app        # Start container in background
podman ps                              # List running containers
podman stop app                        # Stop container
podman rm app                          # Remove container
```

### Flags

- `-t`: Tag the image
- `-d`: Detached mode
- `--name`: Assign a name

---

## 3. Rootless vs Rootful

- **Rootless**:
  - Safer, for users
  - Limited access to privileged ports
- **Rootful**:
  - Full access (e.g., port 80)
  - Use for infrastructure services

Enable lingering for rootless always-on containers:

```bash
loginctl enable-linger <username>
```

---

## 4. Systemd Integration

### Unit Types

| Unit Type | Description |
|-----------|-------------|
| `.service` | Runs a container |
| `.timer` | Runs on a schedule |
| `.target` | Groups services |
| `.socket` | Activates on connection |

### File Locations

- System: `/etc/systemd/system/`
- User: `~/.config/systemd/user/`

### Example `.service`

```ini
[Unit]
Description=FastAPI App
After=network.target

[Service]
ExecStart=/usr/bin/podman start -a fastapi
ExecStop=/usr/bin/podman stop -t 10 fastapi
Restart=on-failure

[Install]
WantedBy=default.target
```

### Commands

```bash
systemctl --user daemon-reload
systemctl --user enable fastapi.service
systemctl --user start fastapi.service
```

---

## 5. Networking: Bridge, Host, veth

### Bridge Network

```bash
podman network create --driver bridge myapp-net
```

- Default network for container-to-container communication

### Host Network

```bash
podman run --network host ...
```

- Shares the host's ports

### veth Pair

- Virtual Ethernet pairs connect containers to host network bridges

---

## 6. Namespaces

| Namespace | Function |
|-----------|----------|
| PID | Process isolation |
| NET | Networking (IP/port) |
| MNT | Mounts |
| UTS | Hostname isolation |
| USER | UID/GID mapping |

```bash
lsns
ls -l /proc/<pid>/ns/
```

---

## 7. Volumes, Mounts

### Named Volume

```bash
podman volume create pgdata
```

### Bind Mount

```bash
podman run -v /host/path:/container/path ...
```

---

## 8. Podman Compose

Used to simulate Docker Compose:
```bash
podman-compose up
```

> Not persistent — use `podman generate systemd` for production.

---

## 9. Podman and Kubernetes

Generate Kubernetes YAML:
```bash
podman generate kube mypod > mypod.yaml
```

Run pods locally:
```bash
podman play kube mypod.yaml
```

---

## 10. Glossary

- **Image**: Blueprint for containers
- **Container**: Instance of an image
- **Namespace**: Kernel resource isolation
- **CNI**: Container Network Interface
- **veth**: Virtual Ethernet device
- **Systemd unit**: Configuration for managing containers

---

## Network and Service Management

### UDS Communication

Use Unix sockets for intra-host app communication:

Backend:
```bash
uvicorn main:app --uds /run/myapp/backend.sock
```

Frontend connects via shared mount:
```bash
-v /run/myapp:/run/myapp
```

---

## DNS Troubleshooting

If DNS fails in rootless builds:

### Solution 1: CLI override

```bash
podman build --dns 8.8.8.8 -t app .
```

### Solution 2: Config file

Edit `~/.config/containers/containers.conf`:

```toml
[network]
dns_servers = ["8.8.8.8"]
```

### Solution 3: Use host networking

```bash
podman build --network host ...
```

---

## CNI Plugins

Used to manage container networks.

### Filesystem Paths

- `/etc/cni/net.d/` (root)
- `~/.config/cni/net.d/` (user)

### Create Network

```bash
podman network create   --driver bridge   --subnet 10.89.0.0/24   --gateway 10.89.0.1   myapp-net
```

---

## End of Guide

This guide contains the key knowledge you need to confidently develop and deploy containerized apps using Podman with systemd, CNI networking, and DNS configuration.
