# Node-RED Settings Documentation

## Overview
This document provides detailed information about the Node-RED `settings.js` configuration file and its various options.

## Core Settings

### Web Server Configuration
- `uiPort`: The port Node-RED web server listens on
  - Default: 1880
  - Can be overridden using PORT environment variable
  - Example: `uiPort: process.env.PORT || 1880`

- `httpAdminRoot`: The root URL for the Node-RED editor
  - Default: '/'
  - Set to false to disable editor
  - Example: `httpAdminRoot: '/admin'`

- `httpStatic`: Directory of static web content
  - Example: `httpStatic: './public'`
  - Useful for serving custom web content

### Security Configuration

#### Admin Authentication
```javascript
adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "$2b$08$your_hashed_password",
        permissions: "*"
    }]
}
```
- `type`: Authentication type
  - "credentials": Username/password authentication
  - "strategy": Custom authentication strategy
- `users`: Array of user objects
  - `username`: Login username
  - `password`: Bcrypt hashed password
  - `permissions`: Access permissions ("*" for full access)

#### Credential Encryption
- `credentialSecret`: Key used to encrypt sensitive data
  - Must be set in production
  - Example: `credentialSecret: "your-secret-key"`

### Editor Configuration

#### Theme Settings
```javascript
editorTheme: {
    page: {
        title: "Node-RED Dashboard",
        favicon: "/path/to/favicon"
    },
    header: {
        title: "Node-RED",
        image: "/path/to/logo"
    }
}
```
- `page`: Customizes the editor page
  - `title`: Browser tab title
  - `favicon`: Custom favicon path
- `header`: Customizes the editor header
  - `title`: Header text
  - `image`: Header logo

#### Palette Management
```javascript
palette: {
    editable: true,
    catalogues: [
        'https://catalogue.nodered.org/catalogue.json'
    ]
}
```
- `editable`: Allow adding/removing nodes via UI
- `catalogues`: Node package catalogs

### File Storage

- `userDir`: Working directory for Node-RED
  - Default: `$HOME/.node-red`
  - Example: `userDir: '/data'`

- `flowFile`: Name of the flow configuration file
  - Default: `flows_<hostname>.json`
  - Example: `flowFile: 'flows.json'`

- `flowFilePretty`: Pretty-print flow JSON
  - Default: false
  - Set to true for better version control

### Logging Configuration
```javascript
logging: {
    console: {
        level: "info",
        metrics: false,
        audit: false
    }
}
```
- `level`: Log level (fatal, error, warn, info, debug, trace)
- `metrics`: Enable/disable metrics logging
- `audit`: Enable/disable audit logging

### Function Node Settings
```javascript
functionGlobalContext: {
    // os: require('os'),
    // moment: require('moment')
}
```
- Defines global objects available in Function nodes
- Can include Node.js modules
- Available as `global.get()` in functions

### Performance Settings

- `nodeMessageBufferMaxLength`: Message buffer size
  - 0 for unlimited
  - Higher values use more memory

- `maxSocketMessageSize`: Maximum WebSocket message size
  - Default: '5mb'
  - Adjust for large messages

### Session Management
```javascript
sessionStorage: {
    type: 'memory',
    options: {
        // path: '/data/sessions'
    }
}
```
- `type`: Storage type (memory, file, memcached)
- `options`: Type-specific configuration

### HTTP Settings

- `apiMaxLength`: Maximum size of HTTP requests
  - Default: '5mb'
  - Adjust for large API calls

- `httpRequestTimeout`: Request timeout in milliseconds
  - Default: 120000 (2 minutes)

### Network Settings

- `socketReconnectTime`: TCP reconnection delay
  - Default: 10000ms (10 seconds)

- `socketTimeout`: Socket timeout
  - Default: 120000ms (2 minutes)

## Best Practices

### Security
1. Always change default credentials
2. Use strong passwords and encryption keys
3. Enable HTTPS in production
4. Restrict editor access if possible
5. Regularly update Node-RED and nodes

### Performance
1. Monitor memory usage
2. Adjust buffer sizes based on needs
3. Enable logging as needed
4. Use appropriate timeout values

### Maintenance
1. Regular backups of flows and settings
2. Version control integration
3. Monitor logs for issues
4. Document custom settings

## Example Configurations

### Basic Development Setup
```javascript
module.exports = {
    uiPort: 1880,
    flowFile: 'flows.json',
    flowFilePretty: true,
    logging: {
        console: {
            level: "debug"
        }
    }
}
```

### Secure Production Setup
```javascript
module.exports = {
    uiPort: process.env.PORT || 1880,
    adminAuth: {
        type: "credentials",
        users: [{
            username: process.env.NODE_RED_USERNAME,
            password: process.env.NODE_RED_PASSWORD,
            permissions: "*"
        }]
    },
    credentialSecret: process.env.NODE_RED_CREDENTIAL_SECRET,
    logging: {
        console: {
            level: "info"
        }
    },
    httpRequestTimeout: 60000,
    maxSocketMessageSize: '2mb'
}
```

## Troubleshooting

### Common Issues
1. Authentication problems
   - Check password hashes
   - Verify credentialSecret
2. File permission issues
   - Check userDir permissions
   - Verify file ownership
3. Memory problems
   - Adjust buffer sizes
   - Monitor resource usage

### Debug Mode
Enable debug logging:
```javascript
logging: {
    console: {
        level: "debug"
    }
}
```
