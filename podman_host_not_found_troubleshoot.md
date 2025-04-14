# Podman: Troubleshooting `Host Not Found` Errors

If you see a `Host not found` error when running or building containers with Podman, it's often caused by DNS resolution issues, misconfigured registries, or missing SSL certificates.

---

## 1. Diagnose the Problem

### Common Scenarios:
- Rootless Podman containers using `slirp4netns` without working DNS
- Internal/private registries unreachable
- Self-signed certificate issues
- Host not allowed in Podman's `registries.conf`

---

## 2. Fix DNS Resolution

### A. Set DNS Manually During Build/Run

```bash
podman build --dns 8.8.8.8 -t myimage .
podman run --dns 10.0.2.2 myimage
```

### B. Persist DNS in Configuration

#### Rootless

```bash
~/.config/containers/containers.conf
```

#### Rootful

```bash
/etc/containers/containers.conf
```

#### Example

```toml
[network]
dns_servers = ["10.0.2.2"]
```

> This ensures all containers use the provided DNS server.

---

## 3. Configure the Registry

### A. Allow Internal/Private Registries

Edit:

```bash
/etc/containers/registries.conf
```

#### Example for insecure registry:

```toml
[[registry]]
prefix = "registry.internal.company"
location = "registry.internal.company"
insecure = true
```

> This disables TLS verification for internal registries.

---

## 4. Add Trusted Self-Signed Certificates

For HTTPS registries with self-signed or internal CA:

```bash
/etc/containers/certs.d/registry.internal.company/ca.crt
```

Make sure the file is named `ca.crt` and the directory name matches the hostname exactly.

---

## 5. Test DNS and Network from Within a Container

Use BusyBox or Alpine to check DNS behavior:

```bash
podman run --rm -it busybox sh
nslookup registry.internal.company
ping google.com
```

---

## 6. Fallback Option: Use Host Networking (Rootful Only)

```bash
sudo podman build --network host -t myimage .
```

This allows the build container to inherit the host’s `/etc/resolv.conf`.

---

## 7. Debug from Inside a Dockerfile

```Dockerfile
RUN apt-get update && apt-get install -y dnsutils
RUN dig registry.internal.company
```

> This helps verify DNS config during image builds.

---

## 8. Summary Checklist

- [ ] Add internal DNS to `containers.conf`
- [ ] Add registry to `registries.conf` with `insecure = true`
- [ ] Place cert in `/etc/containers/certs.d/<host>/ca.crt`
- [ ] Test with `nslookup` or `dig` inside a container
- [ ] Use `--network=host` if all else fails (rootful only)

---

This troubleshooting process will ensure reliable image pulls and container builds in air-gapped or enterprise network environments.
