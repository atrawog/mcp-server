# MCP Fetch Server with Google OAuth - Setup Guide

This guide walks you through setting up a secure MCP fetch server at `mcp-fetch.atradev.org` using Google OAuth 2.0 for authentication and integrating it with Claude.ai.

## Prerequisites

- Domain control for `hexaplant.com`
- Google Workspace account with admin access
- Server with Docker and Docker Compose installed
- Basic knowledge of DNS configuration

## Manual Configuration Steps

### 1. Google Cloud Console Setup

#### Create a New Project
1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Click "Select a project" → "New Project"
3. Name it "MCP Fetch Server" and create

#### Enable Required APIs
1. Go to "APIs & Services" → "Library"
2. Enable the following APIs:
   - Google+ API
   - Google Identity Toolkit API
   - Any other APIs your MCP server might need

#### Configure OAuth Consent Screen
1. Go to "APIs & Services" → "OAuth consent screen"
2. Select **Internal** (for Google Workspace users only)
3. Fill in the required information:
   - **App name**: MCP Fetch Server
   - **User support email**: atrawog@hexaplant.com
   - **Authorized domains**: Add `hexaplant.com`
   - **Developer contact**: atrawog@hexaplant.com
4. Add scopes:
   - `openid`
   - `email` 
   - `profile`
5. Save the configuration

#### Create OAuth 2.0 Credentials
1. Go to "APIs & Services" → "Credentials"
2. Click "Create Credentials" → "OAuth client ID"
3. Choose **Web application**
4. Configure the client:
   ```
   Name: MCP Fetch Server Web Client
   
   Authorized JavaScript origins:
   - https://mcp-fetch.atradev.org
   - https://auth.atradev.org
   
   Authorized redirect URIs:
   - https://auth.atradev.org/oauth/callback
   - https://mcp-fetch.atradev.org/oauth/callback
   ```
5. Click "Create"
6. **Important**: Download the JSON configuration file
7. Save the Client ID and Client Secret for later use

### 2. DNS Configuration

Add the following DNS records to your domain:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | mcp-fetch.atradev.org | YOUR_SERVER_IP | 300 |
| A | auth.atradev.org | YOUR_SERVER_IP | 300 |

Wait for DNS propagation (usually 5-30 minutes).

### 3. Server Preparation

#### Create Directory Structure
```bash
mkdir -p mcp-server/{secrets,config/{redis,oauth-proxy,fetch}}
cd mcp-server
```

#### Prepare Secrets
```bash
# Move the downloaded Google OAuth JSON to secrets
mv ~/Downloads/client_secret_*.json secrets/google_oauth_config.json

# Generate JWT secret
openssl rand -hex 32 > secrets/jwt_secret.txt

# Generate session secret
openssl rand -hex 32 > secrets/session_secret.txt

# Set proper permissions
chmod 600 secrets/*
```

#### Create Environment File
```bash
cat > .env << EOF
# Domain Configuration
DOMAIN=atradev.org
ACME_EMAIL=letsencrypt@dorgeln.org

# Access Control
ALLOWED_EMAILS=atrawog@hexaplant.com
EOF
```

#### Redis Configuration
```bash
cat > config/redis/redis.conf << EOF
# Redis configuration for MCP
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo no
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /data
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
maxmemory 256mb
maxmemory-policy allkeys-lru
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
EOF
```

### 4. Deploy the Stack

```bash
# Clone or create the implementation files
# (See claude.md for implementation details)

# Build and start services
docker-compose build
docker-compose up -d

# Check logs
docker-compose logs -f

# Verify all services are healthy
docker-compose ps
```

### 5. Claude.ai Integration

#### In Claude.ai Settings

1. Go to Claude.ai → Settings → Integrations
2. Add a new custom MCP integration
3. Configure the following:

| Field | Value |
|-------|-------|
| Name | MCP Fetch Server |
| Type | Remote MCP Server |
| Authorization Type | OAuth 2.0 |
| Client ID | YOUR_GOOGLE_CLIENT_ID |
| Client Secret | YOUR_GOOGLE_CLIENT_SECRET |
| Authorization URL | https://auth.atradev.org/oauth/authorize |
| Token URL | https://auth.atradev.org/oauth/token |
| MCP Endpoint | https://mcp-fetch.atradev.org/api/v1/mcp |
| Scopes | fetch:read |

4. Save and test the integration

### 6. Testing and Verification

#### Test OAuth Flow
1. Open a private browser window
2. Go to `https://auth.atradev.org/oauth/authorize`
3. You should be redirected to Google login
4. After login, verify you're redirected back with a token

#### Test MCP Endpoint
```bash
# Get a token first through the OAuth flow
# Then test the MCP endpoint
curl -X POST https://mcp-fetch.atradev.org/api/v1/mcp \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"describe","id":1}'
```

#### Verify SSL Certificates
```bash
# Check auth domain
openssl s_client -connect auth.atradev.org:443 -servername auth.atradev.org

# Check MCP domain
openssl s_client -connect mcp-fetch.atradev.org:443 -servername mcp-fetch.atradev.org
```

### 7. Maintenance

#### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f oauth-proxy
docker-compose logs -f mcp-gateway
docker-compose logs -f caddy
```

#### Update Services
```bash
# Pull latest images
docker-compose pull

# Rebuild and restart
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

#### Backup
```bash
# Backup data volumes
docker run --rm -v mcp-server_redis-data:/data -v $(pwd)/backup:/backup alpine tar czf /backup/redis-data-$(date +%Y%m%d).tar.gz -C /data .
```

## Troubleshooting

### Common Issues

1. **"Invalid redirect URI" error**
   - Ensure the redirect URI in Google Cloud Console matches exactly
   - Check for trailing slashes
   - Verify HTTPS is working correctly

2. **"403 Forbidden" from Google**
   - Ensure OAuth consent screen is configured for Internal use
   - Verify the user email domain is @hexaplant.com
   - Check if the user is in ALLOWED_EMAILS if configured

3. **SSL Certificate issues**
   - Ensure ports 80 and 443 are open
   - Check Caddy logs for Let's Encrypt errors
   - Verify DNS records are correctly configured

4. **"Unauthorized" from MCP endpoint**
   - Check token expiration (tokens are valid for 1 hour)
   - Verify Redis is running and accessible
   - Check oauth-proxy logs for validation errors

### Debug Mode

To enable debug logging:
```bash
# Update .env file
echo "LOG_LEVEL=debug" >> .env

# Restart services
docker-compose restart
```

## Security Notes

1. **Never commit secrets**: The `secrets/` directory should be in `.gitignore`
2. **Regular updates**: Keep Docker images updated for security patches
3. **Monitor logs**: Regularly check logs for suspicious activity
4. **Backup secrets**: Keep secure backups of your secrets files
5. **Rate limiting**: Default is 100 requests/minute per user

## Support

For issues specific to:
- Google OAuth setup: Refer to [Google OAuth documentation](https://developers.google.com/identity/protocols/oauth2)
- MCP Protocol: Check [MCP specification](https://modelcontextprotocol.io)
- Claude.ai integration: Contact Anthropic support

## License

This configuration is provided as-is for use with the MCP fetch server.