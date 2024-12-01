# Node-RED Raspberry Pi Installation and Setup Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Directory Structure Setup](#directory-structure-setup)
4. [Environment Configuration](#environment-configuration)
5. [Service Configuration](#service-configuration)
6. [Settings Configuration](#settings-configuration)
7. [Security Settings](#security-settings)
8. [Service Management](#service-management)
9. [Monitoring](#monitoring)
10. [Troubleshooting](#troubleshooting)

## Prerequisites
- Raspberry Pi with Raspberry Pi OS installed
- User 'raspberry-admin' with sudo privileges
- Internet connection
- Basic terminal knowledge

## Installation

Install Node-RED using the official script:
```bash
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```

## Directory Structure Setup

1. Create the custom directory structure:
```bash
mkdir -p /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
```

2. Set directory permissions:
```bash
chmod 770 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
```

Understanding chmod 770:
- First 7: Owner permissions (raspberry-admin)
  - 4 (read)
  - 2 (write)
  - 1 (execute)
  - Total: 7 (full access)
- Second 7: Group permissions (raspberry-admin group)
  - 4 (read)
  - 2 (write)
  - 1 (execute)
  - Total: 7 (full access)
- Last 0: Others
  - 0 (no access)

## Environment Configuration

1. Generate required credentials:
```bash
# Generate password hash
node-red-admin hash-pw

# Generate credential secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

2. Create .env file:
```bash
cat > /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env << 'EOL'
NODE_RED_USER_DIR=/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
NODERED_USER=admin
NODERED_PASSWORD=$2y$08$yourhashhere
CREDENTIAL_SECRET=yoursecrethere
TZ=Europe/Rome
EOL
```

3. Set proper permissions:
```bash
chmod 600 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env
```

Understanding chmod 600 for .env:
- 6 for owner (raspberry-admin)
  - 4 (read)
  - 2 (write)
  - Total: 6
- 0 for group (no access)
- 0 for others (no access)

## Service Configuration

1. Create or edit the Node-RED service file:
```bash
sudo systemctl edit --full nodered.service
```

2. Add this configuration:
```ini
[Unit]
Description=Node-RED
Wants=network.target
Documentation=http://nodered.org/docs/hardware/raspberrypi.html
After=syslog.target network.target

[Service]
Type=simple
User=raspberry-admin
Group=raspberry-admin
WorkingDirectory=/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
EnvironmentFile=/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env
ExecStart=/usr/bin/node-red-pi --userDir ${NODE_RED_USER_DIR}
Restart=on-failure
RestartSec=10
StartLimitInterval=200
StartLimitBurst=5
KillSignal=SIGINT
TimeoutStartSec=60
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

Important notes:
- Using `EnvironmentFile` instead of individual `Environment` settings
- No need to escape special characters in passwords
- All environment variables are managed in the .env file

## Settings Configuration

1. Create settings.js in your Node-RED directory:
```bash
nano /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/settings.js
```

2. Add this configuration:
```javascript
module.exports = {
    uiPort: process.env.PORT || 1880,
    userDir: process.env.NODE_RED_USER_DIR,
    flowFile: 'flows.json',
    flowFilePretty: true,

    adminAuth: {
        type: "credentials",
        users: [{
            username: process.env.NODERED_USER || 'raspberry-admin',
            password: process.env.NODERED_PASSWORD,
            permissions: "*"
        }]
    },

    credentialSecret: process.env.CREDENTIAL_SECRET,

    logging: {
        console: {
            level: "info",
            metrics: false,
            audit: false
        }
    },

    editorTheme: {
        menu: { "menu-item-help": {
            label: "Node-RED Documentation",
            url: "http://nodered.org/docs"
        } }
    }
}
```

3. Set proper permissions:
```bash
chmod 644 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/settings.js
```

## Security Settings

1. Set proper ownership:
```bash
sudo chown -R raspberry-admin:raspberry-admin /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
```

2. Verify file permissions:
```bash
ls -la /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/
```

## Service Management

1. Enable and start Node-RED:
```bash
# Reload systemd configuration
sudo systemctl daemon-reload

# Enable Node-RED service
sudo systemctl enable nodered.service

# Start Node-RED service
sudo systemctl start nodered.service
```

## Monitoring

1. Check service status:
```bash
# View status
sudo systemctl status nodered

# Monitor logs
journalctl -u nodered -f

# Check environment variables
sudo systemctl show nodered -p Environment
```

2. Verify configuration:
```bash
# Check .env file content
sudo -u raspberry-admin cat /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env

# Check service configuration
sudo systemctl cat nodered
```

## Troubleshooting

### Common Issues

1. Environment Variables Not Loading:
```bash
# Check .env file format
cat -A /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env
# Ensure no spaces around = and no quotes around values
```

2. Permission Issues:
```bash
# Fix .env permissions
chmod 600 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env

# Fix directory permissions
chmod 770 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
```

3. Service Won't Start:
```bash
# Check logs
journalctl -u nodered -n 50 --no-pager

# Verify service file
sudo systemctl cat nodered
```

### Best Practices
1. Always use `EnvironmentFile` in the service configuration
2. Keep environment variables in .env file without quotes
3. Maintain strict file permissions
4. Regular backups of configuration
5. Monitor logs for issues
6. Keep system and Node-RED updated

### Recovery Steps
1. If authentication fails:
```bash
# Generate new password
node-red-admin hash-pw

# Update .env file
nano /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env

# Restart service
sudo systemctl restart nodered
```

2. If credentials are corrupted:
```bash
# Remove credentials file
rm /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/flows_cred.json

# Generate new credential secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Update .env and restart
sudo systemctl restart nodered
```
