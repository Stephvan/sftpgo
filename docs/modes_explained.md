# SFTPGo Operation Modes and Architecture Documentation

## Overview
SFTPGo is a fully featured SFTP server with optional FTP/S and WebDAV support. This document explains the different operation modes and core components.

## Operation Modes

### 1. Standard Mode (serve)

The primary operation mode for production deployments.

#### Usage
```bash
sftpgo serve
```

Key features:
- Full configuration via sftpgo.json
- Multi-user support
- Database backend (SQLite, PostgreSQL, MySQL)
- Virtual folders and quota management
- Detailed logging and monitoring

### 2. Portable Mode

#### Overview
Portable mode is designed for quick, single-directory file sharing without complex configuration.

#### Usage
```bash
sftpgo portable
```

Key features:
- Single directory serving
- Embedded SQLite database
- Simplified configuration
- Ideal for temporary file sharing

### 3. Subsystem Mode

Used when integrating with existing SSH servers.

```bash
sftpgo subsystem
```

## Architecture

### Core Components

1. **Server Core**
   - SFTP protocol implementation
   - Authentication and authorization
   - File system operations
   - Connection management

2. **Data Providers**
   - Database abstraction layer
   - Supports SQLite, PostgreSQL, MySQL
   - User and permissions management
   - Virtual path mapping

3. **Web Interface**
   - Built with Go templates
   - RESTful API backend
   - User management interface
   - File browser and transfer

### Virtual Folders

Virtual folders provide:
- Path mapping and isolation
- Cross-user sharing
- Quota management
- Access control

## Security Features

1. **Authentication Methods**
   - Password
   - Public key
   - Certificate-based
   - Multi-factor authentication

2. **Access Control**
   - Per-user permissions
   - IP filtering
   - Bandwidth throttling
   - Directory restrictions

## Configuration Management

### Main Configuration (sftpgo.json)
```json
{
  "common": {
    "idle_timeout": 15,
    "upload_mode": 2,
    "actions": {
      "execute_on": ["upload", "download", "rename", "delete"]
    }
  },
  "data_provider": {
    "driver": "sqlite",
    "name": "sftpgo.db",
    "host": "",
    "port": 0,
    "username": "",
    "password": ""
  }
}
```

### Database Configuration

#### SQLite (Default)
```json
{
  "driver": "sqlite",
  "name": "sftpgo.db"
}
```

#### PostgreSQL
```json
{
  "driver": "postgresql",
  "name": "sftpgo",
  "host": "localhost",
  "port": 5432,
  "username": "sftpgo",
  "password": "your-password"
}
```

## Deployment

### Production Considerations
1. Database selection based on scale
2. High availability setup
3. Backup strategy
4. SSL/TLS configuration
5. Monitoring setup

### System Requirements
- Memory: 512MB minimum
- CPU: 1 core minimum
- Storage: Depends on usage
- Network: Stable connection

## Monitoring and Maintenance

### Logging
- Detailed activity logs
- Error tracking
- Audit trail
- Performance metrics

### Backup
1. Database backup
2. Configuration backup
3. User data backup
4. Recovery procedures

### Health Checks
- Service status monitoring
- Resource usage tracking
- Connection monitoring
- Error rate tracking

## Server Startup

When you run `sftpgo serve`, the service starts multiple servers based on your configuration:

### Default Services

1. **SFTP Server**
   - Primary SFTP protocol server
   - Default port: 2022 (configurable)
   - Always enabled by default
   - Handles SFTP file transfers and SSH commands

2. **HTTP Server (Web Admin)**
   - Web administration interface
   - REST API endpoints
   - Default port: 8080 (configurable)
   - Enabled by default
   - User management and monitoring

### Optional Services

3. **FTP/S Server**
   - Traditional FTP and FTPS support
   - Disabled by default
   - Requires explicit configuration
   - Supports active and passive modes

4. **WebDAV Server**
   - HTTP-based file transfer
   - Disabled by default
   - Requires explicit configuration
   - Supports SSL/TLS

5. **Telemetry Server**
   - Metrics and monitoring
   - Disabled by default
   - Provides Prometheus metrics
   - Health check endpoints

All servers run concurrently in separate goroutines. You can enable or disable each service in the `sftpgo.json` configuration file. The service will only start servers that are explicitly enabled in the configuration.

## SFTP Implementation Details

The SFTP server implementation is located in the `internal/sftpd` package and provides a full-featured SFTP server with advanced capabilities.

### Core Components

1. **Server Implementation** (`server.go`)
   - Handles incoming SSH connections
   - Manages user authentication
   - Creates SFTP sessions
   - Monitors connection health

2. **SFTP Handler** (`handler.go`)
   - Implements core SFTP protocol operations
   - File operations (read, write, list, delete)
   - Permission management
   - Quota enforcement

### Request Flow

1. **Connection Establishment**
   ```
   Client -> SSH Connection -> Authentication -> SFTP Subsystem
   ```

2. **Session Creation**
   - New SFTP server instance per connection
   - Dedicated handlers for file operations
   - Connection tracking and monitoring

3. **File Operation Handlers**
   - `FileGet`: Read file operations
   - `FilePut`: Write file operations
   - `FileCmd`: File commands (rename, remove)
   - `FileList`: Directory listing

### Features

1. **Security**
   - SSH key authentication
   - Password authentication
   - Permission-based access control
   - IP filtering

2. **File Management**
   - Virtual filesystem support
   - Quota management
   - Directory isolation
   - Path mapping

3. **Monitoring**
   - Activity logging
   - Connection tracking
   - Performance metrics
   - Error reporting

### Integration

The SFTP server integrates with other SFTPGo components:
- User management system
- Virtual filesystem
- Quota management
- Event notifications
- Metrics collection

The implementation uses the `github.com/pkg/sftp` package as the base SFTP protocol implementation while adding enterprise features like user management, permissions, quotas, and monitoring.

## Storage Backend Integration

SFTPGo supports multiple storage backends that can be used as virtual filesystems for SFTP access. Here's how Azure Blob storage integration works:

### Azure Blob Storage

The Azure Blob storage integration (`internal/vfs/azblobfs.go`) allows using Azure Blob containers as SFTP-accessible storage.

#### Configuration

Azure Blob storage can be configured in `sftpgo.json` with the following parameters:

```json
{
  "azblobconfig": {
    "container": "your-container",
    "account_name": "your-storage-account",
    "account_key": "your-account-key",
    "endpoint": "blob.core.windows.net",
    "key_prefix": "folder/",
    "upload_part_size": 5,
    "upload_concurrency": 5,
    "download_part_size": 5,
    "download_concurrency": 5,
    "use_emulator": false
  }
}
```

Key configuration options:
- `container`: Azure Blob container name
- `account_name`: Storage account name
- `account_key`: Storage account key (encrypted)
- `endpoint`: Azure Blob endpoint (default: blob.core.windows.net)
- `key_prefix`: Optional prefix to restrict access to a specific folder
- `upload_part_size`: Size in MB for multipart uploads
- `download_part_size`: Size in MB for multipart downloads

#### Features

1. **Authentication Methods**
   - Account key authentication
   - SAS URL authentication
   - Azure Storage Emulator support

2. **Performance Optimization**
   - Concurrent multipart uploads/downloads
   - Configurable part sizes
   - Connection pooling
   - Buffered operations

3. **Virtual Directory Support**
   - Directory emulation
   - Hierarchical namespace
   - Path prefix restrictions

4. **Integration Features**
   - Quota management
   - Access control
   - Activity monitoring
   - Metrics collection

The Azure Blob storage backend implements the same interface as local storage, making it transparent to SFTP clients while providing cloud storage capabilities.

## HTTP API Integration

SFTPGo provides a comprehensive REST API that allows you to manage all aspects of the service. The API is defined using OpenAPI 3.0 specification in `openapi/openapi.yaml`.

### API Categories

1. **Authentication**
   - `/token` - Get admin access token
   - `/user/token` - Get user access token
   - `/logout` - Invalidate tokens

2. **User Management**
   - User CRUD operations
   - Password management
   - Public key management
   - Two-factor authentication

3. **Storage Management**
   - Virtual folders
   - Quota management
   - File operations
   - Storage backend configuration

4. **System Operations**
   - Connection management
   - Event monitoring
   - IP lists and defender
   - Maintenance operations

### Authentication Methods

1. **API Key Authentication**
   ```
   X-API-Key: your-api-key
   ```

2. **JWT Token Authentication**
   ```
   Authorization: Bearer your-jwt-token
   ```

3. **Basic Authentication**
   ```
   Authorization: Basic base64(username:password)
   ```

### Key Features

1. **RESTful Design**
   - Standard HTTP methods
   - JSON request/response
   - Proper status codes
   - Resource-based URLs

2. **Security**
   - Token-based authentication
   - Role-based access control
   - API key management
   - Rate limiting

3. **Integration Support**
   - OpenAPI specification
   - Swagger documentation
   - Versioned endpoints
   - Consistent error handling

4. **Monitoring**
   - Activity logging
   - Event tracking
   - Quota monitoring
   - Connection status

The HTTP API allows you to automate SFTPGo management and integrate it with other systems while providing a foundation for the web admin interface.

## Headless Operation (API-Only Usage)

SFTPGo can be operated entirely through its REST API without using the web interface. This is particularly useful for:
- Automated deployments
- Custom management interfaces
- System integrations
- CI/CD pipelines

### Configuration

To run SFTPGo in API-only mode, you can configure the HTTP server in `sftpgo.json`:

```json
{
  "httpd": {
    "bindings": [
      {
        "port": 8080,
        "address": "127.0.0.1",
        "enable_web_admin": false,
        "enable_web_client": false
      }
    ]
  }
}
```

### Common API Operations

1. **User Management**
   ```bash
   # Create user
   curl -X POST http://localhost:8080/api/v2/users \
     -H "Authorization: Bearer your-token" \
     -d '{"username":"test", "password":"password"}'

   # Update user
   curl -X PUT http://localhost:8080/api/v2/users/test \
     -H "Authorization: Bearer your-token" \
     -d '{"status":1}'
   ```

2. **Storage Operations**
   ```bash
   # Create virtual folder
   curl -X POST http://localhost:8080/api/v2/folders \
     -H "Authorization: Bearer your-token" \
     -d '{"name":"folder1", "mapped_path":"/path/to/folder"}'
   ```

3. **System Management**
   ```bash
   # Get active connections
   curl http://localhost:8080/api/v2/connections \
     -H "Authorization: Bearer your-token"
   ```

### Automation Benefits

1. **Scriptable Management**
   - Write scripts in any language
   - Automate routine tasks
   - Batch operations
   - Integration testing

2. **Resource Efficiency**
   - No web server overhead
   - Reduced memory usage
   - Minimal CPU usage
   - Lower attack surface

3. **Integration Options**
   - Custom management tools
   - Third-party systems
   - Monitoring solutions
   - Backup systems

4. **Security Advantages**
   - Reduced attack surface
   - Programmatic access control
   - Audit trail
   - Network isolation

The API-first approach allows for complete programmatic control while maintaining all functionality available in the web interface.

## Token Authentication Flow

SFTPGo uses JWT (JSON Web Tokens) for API authentication. Here's how to obtain and use authentication tokens:

### Getting a New Token

1. **Basic Authentication Request**
   - Send a GET request to the `/token` endpoint
   - Include HTTP Basic Authentication header with your username and password
   - If you have 2FA/TOTP enabled, include the passcode in the `X-SFTPGO-OTP` header

   Example:
   ```bash
   curl -X GET "http://your-sftpgo-server/api/v2/token" \
     -H "Authorization: Basic base64(username:password)" \
     -H "X-SFTPGO-OTP: your_totp_code"  # Only if 2FA is enabled
   ```

2. **Server Authentication Flow**
   - The server validates your username and password against the configured data provider
   - If 2FA is enabled, it validates the TOTP passcode
   - The server checks IP address restrictions and other security policies
   - If authentication succeeds, it generates a JWT token

3. **Token Response**
   - On successful authentication, you receive a JSON response containing:
     ```json
     {
       "access_token": "your.jwt.token"
     }
     ```
   - The token is signed using HMAC-SHA256 (HS256)
   - The token contains claims like username, permissions, and expiration time

4. **Using the Token**
   - For subsequent API requests, include the token in the Authorization header:
     ```
     Authorization: Bearer your.jwt.token
     ```
   - The token is valid for a limited time period
   - When the token expires, you need to request a new one

### Token Types

SFTPGo supports two types of tokens:

1. **Admin Tokens**
   - Obtained by authenticating with admin credentials
   - Have full access to admin API endpoints
   - Can manage users, view server status, etc.

2. **User Tokens**
   - Obtained by authenticating with regular user credentials
   - Limited to user-specific operations
   - Can access web client features and user-specific API endpoints

## HTTP Server and API

SFTPGo runs a single HTTP server that serves multiple features. Here's how it works:

### Server Configuration

The HTTP server is configured in the `httpd` section of `sftpgo.json`:
```json
{
  "httpd": {
    "bindings": [
      {
        "port": 8080,
        "address": "",
        "enable_web_admin": true,
        "enable_web_client": true,
        "enable_rest_api": true
      }
    ]
  }
}
```

### Features and Components

The HTTP server provides multiple features through a single server instance:

1. **REST API**
   - Enabled via `enable_rest_api`
   - All API endpoints under `/api/v2/*`
   - Authentication via `/api/v2/token`
   - Health check at `/healthz`

2. **Web Admin Interface**
   - Enabled via `enable_web_admin`
   - Management interface for administrators
   - User management, monitoring, and configuration

3. **Web Client Interface**
   - Enabled via `enable_web_client`
   - Web-based file management for end users
   - File sharing and personal settings

4. **OpenAPI Documentation**
   - Interactive API documentation
   - Available when enabled in configuration

### Server Modes

You can run the HTTP server in different modes by enabling/disabling features:

1. **Full Mode**
   - All features enabled (API, web admin, web client)
   - Default configuration

2. **API-Only Mode**
   ```json
   {
     "httpd": {
       "bindings": [
         {
           "enable_web_admin": false,
           "enable_web_client": false,
           "enable_rest_api": true
         }
       ]
     }
   }
   ```

3. **Admin-Only Mode**
   - Disable web client but keep API and admin interface
   - Useful for administrative setups

4. **Client-Only Mode**
   - Disable web admin but keep API and web client
   - Suitable for end-user focused deployments

### Server Startup

When you run `sftpgo serve`, the following servers are started based on your configuration:

1. **Core Servers** (Always Started)
   - SFTP Server (default port 2022)
   - HTTP Server (default port 8080)

2. **Optional Servers** (Started if Enabled)
   - FTP/S Server
   - WebDAV Server
   - Telemetry Server

Each server runs in its own goroutine, allowing for concurrent operation of all enabled services.
