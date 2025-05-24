# MCP Fetch Server Implementation Requirements

This document outlines the requirements and architecture for implementing a secure MCP fetch server with Google OAuth 2.0 authentication. The implementation should be created using Docker Compose and designed for production deployment.

## Overview

Create a production-ready MCP fetch server deployment that:
- Runs at `mcp-fetch.atradev.org`
- Uses Google OAuth 2.0 for authentication (not self-hosted OAuth)
- Integrates with Claude.ai as a remote MCP server
- Supports enterprise Google Workspace accounts
- Provides a scalable authentication solution for multiple services

## Key Constraints

### Google OAuth Limitations
- **No wildcard domain support**: Cannot use `*.hexaplant.com` as authorized origin
- Each subdomain must be explicitly registered in Google Cloud Console
- Must implement a workaround for multi-service authentication

### Security Requirements
- MCP servers must implement OAuth 2.1 with PKCE for Claude.ai integration
- All endpoints must use valid TLS certificates (no self-signed)
- Must prevent access to internal networks from fetch requests
- Implement proper CORS headers for Claude.ai access

## Architecture Requirements

### Service Components

1. **Reverse Proxy (Caddy)**
   - Automatic HTTPS with Let's Encrypt
   - Route `/api/*` to MCP gateway
   - Route `/oauth/*` to OAuth proxy
   - Security headers implementation

2. **OAuth Proxy Service**
   - Handle Google OAuth 2.0 flow
   - Restrict to @hexaplant.com domain users
   - Issue internal JWT tokens (not Google tokens directly)
   - Implement token refresh mechanism
   - Session management with Redis

3. **MCP Gateway**
   - Validate JWT tokens with OAuth proxy
   - Rate limiting per user (100 req/min)
   - Forward validated requests to MCP fetch server
   - Add user context headers
   - Block internal network URLs

4. **MCP Fetch Server**
   - Run actual mcp-server-fetch
   - Isolated from direct internet access
   - Only accessible through gateway

5. **Redis**
   - Session storage
   - Rate limiting counters
   - Token blacklist/revocation

### Network Architecture
- Two networks: internal (isolated) and external
- MCP fetch server only on internal network
- Gateway bridges internal/external networks
- Redis only on internal network

## Implementation Specifications

### OAuth Proxy Requirements

The OAuth proxy must:
1. Implement Google OAuth 2.0 web flow
2. Validate email domain is @hexaplant.com
3. Support configurable email allowlist
4. Issue JWT tokens with:
   - 1-hour expiration
   - User email and ID
   - Scope: `fetch:read`
   - Issuer: `https://auth.hexaplant.com`
5. Provide endpoints:
   - `/oauth/authorize` - Start OAuth flow
   - `/oauth/callback` - Handle Google callback
   - `/oauth/validate` - Validate JWT tokens
   - `/oauth/refresh` - Refresh tokens
   - `/health` - Health check

### MCP Gateway Requirements

The gateway must:
1. Validate every request's JWT token
2. Implement rate limiting using Redis
3. Block requests to internal IPs/localhost
4. Forward user context to fetch server
5. Handle CORS for Claude.ai domains
6. Provide proper error responses

### Security Implementation

1. **Container Security**
   - Run as non-root user (1000:1000)
   - Read-only root filesystem
   - Drop all capabilities
   - No new privileges flag

2. **Network Security**
   - Internal services not exposed
   - All external traffic through Caddy
   - Validate all input URLs
   - Prevent SSRF attacks

3. **Token Security**
   - Short-lived tokens (1 hour)
   - Refresh token rotation
   - Token revocation support
   - Secure secret management

## Multi-Service Authentication Strategy

Since Google doesn't support wildcard domains, implement:

1. **Central Auth Domain** (`auth.hexaplant.com`)
   - All OAuth flows go through this domain
   - Register once in Google Cloud Console
   - Issue tokens valid for all services

2. **Service Integration Pattern**
   - Services redirect to auth domain
   - Pass target service in state parameter
   - Auth domain redirects back with token
   - Services validate token with auth proxy

3. **Shared Session Strategy**
   - Use Redis for cross-domain sessions
   - JWT tokens as portable credentials
   - Same auth proxy validates all services

## Environment Configuration

Required environment variables:
```
DOMAIN=hexaplant.com
ACME_EMAIL=admin email for Let's Encrypt
ALLOWED_EMAILS=comma-separated list or empty for domain-only
GOOGLE_CLIENT_CONFIG_FILE=/run/secrets/google_oauth_config
JWT_SECRET_FILE=/run/secrets/jwt_secret
SESSION_SECRET_FILE=/run/secrets/session_secret
REDIS_URL=redis://redis:6379
LOG_LEVEL=info|debug
```

## File Structure

```
mcp-fetch-server/
├── docker-compose.yml
├── .env
├── Dockerfile.oauth-proxy
├── Dockerfile.gateway
├── Dockerfile.fetch
├── Caddyfile
├── oauth-proxy.js (or Python equivalent)
├── mcp_gateway.py (or JS equivalent)
├── requirements.txt / package.json
├── config/
│   ├── redis/
│   │   └── redis.conf
│   └── ... other configs
└── secrets/
    ├── google_oauth_config.json
    ├── jwt_secret.txt
    └── session_secret.txt
```

## Testing Requirements

Implement health checks for all services:
- OAuth proxy: `/health` endpoint
- MCP gateway: `/health` endpoint  
- MCP fetch: Standard health check
- Redis: `redis-cli ping`

## Deployment Considerations

1. **Resource Limits**
   - MCP fetch: 512MB RAM, 0.5 CPU
   - OAuth proxy: 256MB RAM, 0.25 CPU
   - Gateway: 256MB RAM, 0.25 CPU
   - Redis: 256MB RAM, 0.25 CPU

2. **Persistent Storage**
   - Redis data volume for sessions
   - Caddy data volume for certificates
   - MCP data volume if needed

3. **Logging**
   - Structured JSON logging
   - Log aggregation ready
   - No sensitive data in logs

## Error Handling

Implement proper error responses:
- 401 Unauthorized: Invalid/missing token
- 403 Forbidden: Invalid domain/email
- 429 Too Many Requests: Rate limit exceeded
- 400 Bad Request: Invalid URL/parameters
- 500 Internal Server Error: Service failures

## Claude.ai Integration Points

The implementation must provide:
1. OAuth 2.0 authorization endpoint
2. OAuth 2.0 token endpoint  
3. MCP-compliant API endpoint
4. Proper CORS headers for browser access
5. MCP protocol version headers

## Success Criteria

The implementation is complete when:
1. Google OAuth login works for @hexaplant.com users
2. Claude.ai can successfully authorize and connect
3. MCP fetch operations work through the gateway
4. All security measures are in place
5. Services auto-restart on failure
6. HTTPS works automatically
7. Logs are clean and informative

## Additional Considerations

- Consider implementing webhook support for OAuth events
- Plan for monitoring and alerting integration
- Design for horizontal scaling if needed
- Document any assumptions or limitations
- Provide clear error messages for debugging