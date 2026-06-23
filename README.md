# Bline-X Agent

The mesh VPN agent for [Bline-X](https://github.com/DJR-FP/blinex) — connects your devices into a secure WireGuard mesh network.

## Download

Pre-built binaries for each release:

| Platform | Binary |
|----------|--------|
| Linux (amd64) | `blinex-agent-linux-amd64` |
| Linux (arm64) | `blinex-agent-linux-arm64` |
| macOS (Apple Silicon) | `blinex-agent-darwin-arm64` |
| macOS (Intel) | `blinex-agent-darwin-amd64` |
| Windows (amd64) | `blinex-agent-windows-amd64.exe` |
| Windows (arm64) | `blinex-agent-windows-arm64.exe` |

Download from the [Releases](https://github.com/DJR-FP/blinex-agent/releases) page.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/DJR-FP/blinex-agent/main/install.sh | \
  BLINEX_SETUP_KEY=YOUR_KEY \
  BLINEX_MANAGEMENT_URL=your-server:50051 \
  BLINEX_SIGNAL_URL=your-server:10000 \
  BLINEX_TURN_PASS=your-relay-password \
  sudo -E bash
```

| Variable | Default | Description |
|----------|---------|-------------|
| `BLINEX_SETUP_KEY` | _(required)_ | Enrollment key from the Setup Keys page |
| `BLINEX_MANAGEMENT_URL` | `localhost:50051` | Management server gRPC address |
| `BLINEX_SIGNAL_URL` | `localhost:10000` | Signal server address |
| `BLINEX_TURN_PASS` | _(empty)_ | TURN relay password (must match `RELAY_AUTH_PASS` on server) |
| `BLINEX_TURN_USER` | `blinex` | TURN relay username |
| `BLINEX_RELAY_URL` | _(auto-detected)_ | TURN relay host:port (defaults to management host:3478) |
| `BLINEX_VERSION` | `latest` | Pin a specific release version |

This downloads the agent, writes a config file, and starts a systemd service (Linux) or launchd daemon (macOS).

## Uninstall

**Linux / macOS:**

```bash
curl -fsSL https://raw.githubusercontent.com/DJR-FP/blinex-agent/main/uninstall.sh | sudo bash
```

Or download the uninstall binary from the [latest release](https://github.com/DJR-FP/blinex-agent/releases):

```bash
curl -fsSL https://github.com/DJR-FP/blinex-agent/releases/latest/download/blinex-uninstall-linux-amd64 -o blinex-uninstall
chmod +x blinex-uninstall
sudo ./blinex-uninstall
```

**Windows:** Download `blinex-uninstall-windows-amd64.exe` from the [latest release](https://github.com/DJR-FP/blinex-agent/releases) and run as Administrator.

### What gets removed

| Platform | Removed |
|----------|---------|
| Linux | systemd service, `/usr/local/bin/blinex-agent`, `/etc/blinex/`, `/var/lib/blinex/`, `BLINEX-ACL` iptables chain, `blinex0` interface |
| macOS | launchd service, binary, config, state, log file |
| Windows | `BlinexAgent` service, `%ProgramFiles%\Bline-X\`, `%ProgramData%\Bline-X\`, firewall rules, PATH entry |

The device stays listed in the dashboard until you remove it there.

## Running in an LXC container

Unprivileged LXC containers have no `/dev/net/tun`, so the agent falls back to userspace netstack mode — inbound works, but the container's own apps can't reach the mesh transparently. Pass the TUN device through for full connectivity. On the Proxmox host:

```bash
echo "lxc.cgroup2.devices.allow: c 10:200 rwm" >> /etc/pve/lxc/<CTID>.conf
echo "lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file" >> /etc/pve/lxc/<CTID>.conf
pct restart <CTID>
```

Then reinstall the agent — it will detect the TUN device and use kernel mode.

## Documentation

Full docs at the main repo: [DJR-FP/blinex](https://github.com/DJR-FP/blinex)
