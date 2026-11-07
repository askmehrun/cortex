# systemd Services

Run every long-lived process as a systemd unit so it survives reboots and crashes. A unit file at `/etc/systemd/system/<name>.service` declares the process, its dependencies, and restart policy.

## When to Use
- Any service that must be running all the time (web servers, databases, containers).
- Anything you previously started by hand or with a shell script.
- A process that dies when its terminal closes.

## How
```ini
[Unit]
Description=Service Name
After=docker.service network-online.target
Requires=docker.service

[Service]
Type=simple
Restart=always
RestartSec=5
ExecStartPre=-/usr/bin/docker stop <container>
ExecStartPre=-/usr/bin/docker rm <container>
ExecStart=/usr/bin/docker run --name <container> <args...>
ExecStop=/usr/bin/docker stop <container>

[Install]
WantedBy=multi-user.target
```

Key fields:
- `After=` — ordering dependency (don't start before X).
- `Requires=` — hard dependency (fail if X is down).
- `Restart=always` — auto-restart on crash.
- `ExecStartPre=-` — prefix `-` ignores the command's failure.
- `WantedBy=multi-user.target` — enable on boot.

```bash
systemctl daemon-reload      # reload unit files
systemctl enable <name>      # start on boot
systemctl start <name>       # start now
systemctl status <name>      # check status
journalctl -u <name> -f      # live logs
systemctl restart <name>     # restart
```

## Rule
Never run long-lived processes in a terminal. Always systemd.
