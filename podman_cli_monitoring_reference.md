## Podman CLI Monitoring and Control Reference

This section is the CLI-based complement to the systemd-based lifecycle. It's intended for use during development, debugging, or simple rootless user applications that don't require full orchestration via `systemctl`.

---

### Monitor and Control Containers with Podman CLI

| **Task**                        | **Podman Command**                                  | **Notes** |
|----------------------------------|------------------------------------------------------|-----------|
| Start container                 | `podman start <container>`                          | Resumes existing container |
| Stop container                  | `podman stop <container>`                           | Sends SIGTERM, then SIGKILL |
| Run container                   | `podman run --name <container> <image>`             | Creates + starts new container |
| Remove container                | `podman rm <container>`                             | Use `-f` to force if running |
| Remove image                    | `podman rmi <image>`                                | Clean unused layers |
| View logs                       | `podman logs <container>`                           | Works for current container run |
| Follow live logs                | `podman logs -f <container>`                        | Like `tail -f` |
| Inspect container (full JSON)   | `podman inspect <container>`                        | Deep configuration view |
| View running containers         | `podman ps`                                         | Add `-a` to show stopped ones |
| Check image history             | `podman history <image>`                            | Show layer history |
| View container resource usage   | `podman stats`                                      | Like `top` for containers |
| List images                     | `podman images`                                     | Show available local images |
| Prune stopped containers/images | `podman system prune`                               | Safely clears unused containers/images/networks |
| Re-enter container shell        | `podman exec -it <container> sh` or `bash`          | Useful for debugging |

---

## Best Practices for Podman (Non-systemd)

1. **Use `--name` for all containers**:
   - Makes referencing easier across logs, networks, and volumes.

2. **Use tagged image names (`-t`) when building**:
   ```bash
   podman build -t my-api .
   ```

3. **Always use volumes for data persistence**:
   ```bash
   podman run -v pgdata:/var/lib/postgresql/data ...
   ```

4. **Check `podman inspect` to debug configuration or health checks**.

5. **Leverage custom networks** to isolate app stacks:
   ```bash
   podman network create myapp-net
   podman run --network myapp-net ...
   ```

6. **Use `podman generate systemd` to transition to persistent services** once tested and stable.

7. **Run logs through `journalctl` only after `systemctl` is used** — otherwise rely on `podman logs`.

8. **Clean up often** with:
   ```bash
   podman ps -a
   podman system prune
   ```

9. **Mount config/certs intentionally**:
   - Use `--volume` or `-v` to mount app config or secrets.
   - Use `/etc/containers/certs.d/<registry>/ca.crt` for internal HTTPS registries.

---

