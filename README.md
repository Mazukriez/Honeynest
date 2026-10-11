# Honeynest
a self-contained project called Honeynest: Docker-based, with several honeypots, a central dashboard and an installer.
I'll build this as a self-contained project called **Honeynest**: Docker-based, with several honeypots, a central dashboard and an installer. First I'll check what the sandbox has so I can test as I go.

I built **Honeynest**, a multi-honeypot platform with an installer and a built-in dashboard. The Docker images were never built or run, because this sandbox has no Docker daemon. The compose file validates, but you'll be the first to start the real containers. I tested everything else as running processes. Install steps are in `README.md`:

```bash
tar xzf honeynest-1.0.0.tar.gz && cd honeynest-1.0.0
sudo ./install.sh        # generates a dashboard password, builds and starts everything
```

- **Honeypots:** SSH and Telnet drop attackers into a fake shell, HTTP/HTTPS present fake WordPress/phpMyAdmin/Tomcat logins, and FTP, SMTP, Redis and MySQL capture credentials and commands. A catch-all records the first bytes of probes to SMB, RDP, VNC and other ports. They're written from scratch (stdlib asyncio plus asyncssh), not wrappers around Cowrie or Dionaea.
- **Dashboard:** The dashboard has live events, an activity timeline, top attackers, credentials, commands and paths, per-attacker drill-down, search, CSV/NDJSON export, Prometheus metrics and optional webhook alerts. It listens on loopback only by default, behind a login, and works offline.
- **Safety:** Nothing an attacker sends is ever executed, and attacker URLs are logged, never downloaded. Containers run non-root with a read-only filesystem and no capabilities. `scripts/egress-lockdown.sh` blocks outbound traffic from the honeypot network.

**What I tested:**
- 28 automated tests pass, including fuzzing every service with malformed input. They also pass from the extracted tarball.
- I ran the real entrypoint and dashboard together and attacked them over SSH, HTTPS and Redis.
- I loaded the UI in headless Chromium with XSS payloads injected: none executed, and there were no console errors.
- The installer ran against a stubbed `docker`.

Along the way, testing caught two real bugs. Download URLs were being logged twice, and requests with unencoded spaces in the path (common in exploit scanners) were misclassified as non-HTTP. Both are fixed.

**Differences from T-Pot:**
- It captures credentials, commands, exploit attempts and botnet traffic well, but not malware binaries, since it never downloads anything.
- It is lower-interaction than T-Pot, so a skilled human may eventually spot the emulation.
- The egress lockdown covers IPv4 only.

Run it on a dedicated VPS or VM, not next to anything you care about. To catch real SSH attacks on port 22, move your own admin SSH to another port first; the README walks through it safely.

The `.github/workflows/ci.yml` file includes a Docker build and smoke test, and I haven't run it.

Files are in `/home/claude/honeynest`:
- honeynest-1.0.0.tar.gz
