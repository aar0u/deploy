# browser-box

A pure headless browser & desktop runtime base image designed for container multi-tenant automation and scrapers.

Published to `ghcr.io/<owner>/browser-box:latest`.

## What's Inside

- **Node.js**: v22 LTS (slim)
- **Virtual Display**: Xvfb (`:99`) + Fluxbox window manager
- **Remote Desktop**: x11vnc (port 5900) + noVNC web interface (port 8445)
- **Browser**: Playwright Chromium pre-installed at `/ms-playwright` (`PLAYWRIGHT_BROWSERS_PATH`)
- **Process Supervisor**: `supervisord` as PID 1 with automatic zombie reaping and service auto-restart

## Architecture: Landlord & Tenants (房东与租客)

`browser-box` contains **zero application code**. It acts as the "landlord" providing infrastructure.

Tenant applications (e.g., `catvodspiderjs`, `signal-hub`) mount under `/apps/` and define their own `*.supervisor.conf` files. Supervisord automatically discovers and manages them.

### Directory Layout on Host

```text
/home/azureuser/apps/
├── catvodspiderjs/
│   ├── package.json
│   ├── proxy/
│   └── catvod.supervisor.conf
└── signal-hub/
    ├── package.json
    ├── actsg/
    └── signal.supervisor.conf
```

### Example Tenant Configuration (`catvod.supervisor.conf`)

```ini
[program:catvod-proxy]
directory=/apps/catvodspiderjs/proxy
command=npm start
environment=PORT="8787",BROWSER_TIMEOUT="1200",PAGE_TIMEOUT="120",PAGE_CLOSE_DELAY="2",DISPLAY=":99",HEADLESS="false"
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/catvod.log
stderr_logfile=/var/log/supervisor/catvod.err
```

### Running the Container

```bash
docker run -d \
  --name browser-box \
  --restart=unless-stopped \
  --init \
  --shm-size=1g \
  --network my_internal_net \
  -p 8445:8445 \
  -v /home/azureuser/apps:/apps \
  ghcr.io/<owner>/browser-box:latest
```

### Management via Supervisor

```bash
# Check status of all base services and tenant apps
docker exec browser-box supervisorctl status

# Restart a specific tenant app
docker exec browser-box supervisorctl restart catvod-proxy

# View logs in real time
docker exec browser-box supervisorctl tail -f catvod-proxy
```
