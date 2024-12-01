# Node-RED Raspberry Pi Installation and Setup Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Directory Structure Setup](#directory-structure-setup)
4. [Service Configuration](#service-configuration)
5. [Environment Configuration](#environment-configuration)
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

## Service Configuration

1. Create or edit the Node-RED service file:
```bash
sudo systemctl edit --full nodered.service
```

2. Add the following configuration:
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
Environment="NODE_RED_USER_DIR=/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered"
Environment="NODE_RED_ENV_FILE=/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env"
ExecStart=/usr/bin/node-red-pi --userDir ${NODE_RED_USER_DIR}

# Enhanced service monitoring
Restart=on-failure
RestartSec=10
StartLimitInterval=200
StartLimitBurst=5

# Process management
KillSignal=SIGINT
TimeoutStartSec=60
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

## Environment Configuration

1. Create .env file:
```bash
nano /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env
```

2. Add environment variables:
```plaintext
# Node-RED Directory and Configuration
NODE_RED_USER_DIR='/home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered'

# Node-RED Credentials
NODERED_USER='raspberry-admin'
NODERED_PASSWORD='your_hashed_password'
CREDENTIAL_SECRET='your_generated_secret'

# Timezone
TZ='Europe/Rome'
```

3. Set .env file permissions:
```bash
chmod 600 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env
```

## Settings Configuration

1. Create settings.js in your Node-RED directory:
```bash
nano /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/settings.js
```

2. Add the following configuration:
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

2. Generate password hash:
```bash
# Using Node-RED command
node-red-admin hash-pw
```

3. Generate credential secret:
```bash
# Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
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

# Check restart count
systemctl show nodered --property=NRestarts
```

2. Monitor service health:
```bash
# Check if process is running
ps aux | grep node-red

# Check port
sudo netstat -tulpn | grep 1880

# Check system resources
top -u raspberry-admin
```

## Troubleshooting

### Common Issues

1. Service won't start:
```bash
# Check logs for errors
journalctl -u nodered -n 50 --no-pager

# Verify permissions
ls -la /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered

# Check configuration
sudo systemctl cat nodered
```

2. Authentication issues:
```bash
# Verify .env file
cat /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered/.env

# Check environment variables
sudo systemctl show nodered -p Environment

# Regenerate password if needed
node-red-admin hash-pw
```

3. Directory issues:
```bash
# Verify directory exists and has correct permissions
ls -la /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered

# Fix permissions if needed
sudo chown -R raspberry-admin:raspberry-admin /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
chmod 770 /home/raspberry-admin/poggiolo-salone/poggiolo-salone-nodered/nodered
```

### Best Practices
1. Regular backups of flows and configurations
2. Monitor system logs
3. Keep Node-RED and system updated
4. Regular security audits
5. Document any custom modifications
6. Test changes in a safe environment first
