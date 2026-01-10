# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-10

### Added
- **Rate Limiting**: Added express-rate-limit middleware
  - General rate limit: 100 requests per 15 minutes per IP
  - Proxy-specific rate limit: 50 requests per 15 minutes per IP
  - Rate limit headers in responses (RateLimit-Limit, RateLimit-Remaining, RateLimit-Reset)
- **SSRF Protection**: Enhanced URL validation to block:
  - Localhost (localhost, 127.0.0.1, 0.0.0.0)
  - Private IP ranges (10.x.x.x, 172.16-31.x.x, 192.168.x.x)
- **Security Headers**: Enhanced Helmet.js configuration with:
  - Content Security Policy (CSP)
  - HTTP Strict Transport Security (HSTS)
  - X-Content-Type-Options (nosniff)
  - Referrer-Policy
  - Permitted Cross-Domain Policies
- **Docker Support**: 
  - Production-ready Dockerfile with non-root user
  - Health check configuration
  - docker-compose.yml for easy deployment
  - .dockerignore for optimized builds
- **Documentation**:
  - Comprehensive DEPLOYMENT.md guide
  - Updated SECURITY.md with proper vulnerability reporting
  - Enhanced README.md with security features and deployment options
  - Production deployment checklist

### Changed
- **Error Handling**: Improved error messages to prevent information leakage in production
  - Development mode: Shows detailed error messages
  - Production mode: Shows generic error messages
- **URL Validation**: Enhanced with sanitization to remove credentials from URLs
- **Environment Configuration**: Expanded .env.example with all security options
- **.gitignore**: Uncommented .env to prevent committing sensitive data

### Security
- **Fixed npm vulnerabilities**:
  - Updated express from 4.18.2 to 4.21.2
  - Updated body-parser to resolve qs vulnerability
  - Updated js-yaml to fix prototype pollution
- **Removed .env from git tracking**: Prevents exposure of sensitive configuration
- **Input Validation**: Stricter URL validation and sanitization
- **Request Size Limits**: Enforced 10MB limit for request bodies
- **CodeQL**: Passed security scan with 0 vulnerabilities

### Fixed
- Potential SSRF attacks through URL validation
- Information disclosure through verbose error messages
- Missing security headers
- Untracked .env file containing sensitive data

## [0.x.x] - Before 2026-01-10

### Features
- Native Node.js HTTP/HTTPS proxy
- File download with streaming support
- WebSocket proxy support
- Multiple HTTP methods support (GET, POST, PUT, DELETE, PATCH)
- SSL certificate bypass for development
- Range request support for large files
- Time endpoint with greeting
- Health check endpoint
- Path-based API routing
- Vercel deployment configuration

---

## Upgrade Guide

### From 0.x.x to 1.0.0

1. **Update dependencies:**
   ```bash
   npm install
   ```

2. **Update environment variables:**
   - Copy new variables from `.env.example` to your `.env`
   - Set `NODE_ENV=production` for production deployments
   - Configure `CORS_ORIGIN` to specific domains (not `*`)

3. **Review rate limits:**
   - Default: 100 requests per 15 minutes
   - Proxy: 50 requests per 15 minutes
   - Adjust if needed in `.env`

4. **Update deployment configuration:**
   - Use new Dockerfile if deploying with Docker
   - Update nginx configuration if using reverse proxy

5. **Test SSRF protection:**
   - Private IPs and localhost are now blocked
   - Ensure your use case doesn't require these (they shouldn't)

### Breaking Changes

- **Private IP blocking**: Requests to localhost and private IP ranges are now blocked by default
  - This is a security feature and should not be disabled
  - If you need to proxy to internal services, consider alternative architectures

### Deprecations

None in this release.
