# Safely Managing PostgreSQL with Podman Volumes (Dev + Systemd)

This guide shows you how to properly manage your PostgreSQL data using **Podman volumes**, with workflows for both **development** and **production (systemd)**. It includes how to avoid data loss, how volumes work, and why this is better than relying on `.sql` dump injection alone.

---

## Why This Guide?

You may be transitioning from Docker Compose or `.sql` dump workflows where:
- You inject `.sql` into the container each time
- You lose changes after deleting containers
- You "reset" state manually and risk overwriting schema/data

With Podman volumes, **your data survives reboots, rebuilds, and crashes**.

---

## Quick TL;DR

**Always use:**
```bash
-v pgdata:/var/lib/postgresql/data
```
- This is a **named volume**
- It stores **all PostgreSQL database files**
- It **persists** across container rebuilds and reboots

---

## 1. Understanding Volumes vs Dumps

| Feature | `.sql` Dumps | Podman Volumes |
|--------|--------------|----------------|
| One-time schema/data import | Yes | No |
| Changes persist across rebuilds | No | **Yes** |
| Supports ongoing dev work | No | **Yes** |
| Safer during crashes | No | **Yes** |
| Default in production | No | **Yes** |

---

## 2. How Volumes Work

Volumes are stored outside the container:

- **Rootless**: `$HOME/.local/share/containers/storage/volumes/pgdata`
- **Rootful**: `/var/lib/containers/storage/volumes/pgdata`

The container mounts it into `/var/lib/postgresql/data`, where Postgres stores all data.

---

## 3. The Right Way (Dev + Volumes)

### Step 1: Create a named volume

```bash
podman volume create pgdata
```

### Step 2: Optional — Seed DB with SQL Dump

```bash
podman run --rm   -v pgdata:/var/lib/postgresql/data   -v $(pwd)/init.sql:/docker-entrypoint-initdb.d/init.sql   -e POSTGRES_PASSWORD=secret   postgres:15
```

### Step 3: Run container with persistent volume

```bash
podman run -d   --name db   -v pgdata:/var/lib/postgresql/data   -e POSTGRES_PASSWORD=secret   postgres:15
```

Postgres writes to the volume. Changes are saved.

---

## 4. Confirming Volume Behavior

- Add schema, rows, roles, etc.
- Stop and remove the container:
  ```bash
  podman stop db && podman rm db
  ```
- Start it again:
  ```bash
  podman run -d --name db -v pgdata:/var/lib/postgresql/data postgres:15
  ```
- **All your data is still there.**

---

## 5. Avoiding the Pitfall You Had with SQL Dumps

**Problem you faced:**
- Removed cache or rebuilt containers
- DB reverted to last `.sql` snapshot
- Schema changes disappeared

**Why it happened:**
- You were re-seeding from static SQL files
- No volume was preserving real data changes

**Solution with volumes:**
- Your schema and data evolve *in-place*
- You don’t need to reseed unless you want to reset manually
- Even crashes won't lose your data

---

## 6. What About `podman system prune`?

```bash
podman system prune
```
- Removes stopped containers, unused images
- **Does NOT delete named volumes**
- **Your pgdata volume is safe**

---

## 7. Transition to Production with systemd

### Generate unit file:

```bash
podman generate systemd --name db --files --restart-policy=always
```

### Move and enable:

```bash
mv container-db.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-db
```

### Benefits:
- Auto-starts on boot/login
- Restarts on crash
- Still uses the same named volume for data persistence

---

## 8. Backup and Restore

### Backup:

```bash
podman run --rm -v pgdata:/data alpine tar czf - -C /data . > pgdata.tar.gz
```

### Restore:

```bash
cat pgdata.tar.gz | podman run --rm -i -v pgdata:/data alpine tar xzf - -C /data
```

---

## 9. Summary Workflow (Dev to Prod)

| Task | Command |
|------|---------|
| Create volume | `podman volume create pgdata` |
| Run container | `podman run -v pgdata:/var/lib/postgresql/data postgres:15` |
| Reset volume | `podman volume rm pgdata && podman volume create pgdata` |
| Backup volume | `tar czf` in temp container |
| systemd service | `podman generate systemd` |
| Inspect volume | `podman volume inspect pgdata` |

---

## Final Advice

- **Stop using `.sql` dumps** as your daily data store
- **Mount volumes** and let Postgres manage real state
- **Inspect and back up** regularly
- **Use systemd** once stable

This way, your database behaves like a real, durable service — not a temp script in a container.
