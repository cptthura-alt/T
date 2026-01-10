# Production Deployment Guide

This guide covers deploying the secure proxy server to production environments.

## Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd T
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Edit .env with your settings
   ```

4. **Start the server**
   ```bash
   npm start
   ```

## Deployment Options

### 1. Docker Deployment (Recommended)

**Build and run:**
```bash
docker build -t secure-proxy-server .
docker run -d -p 3000:3000 \
  -e NODE_ENV=production \
  -e CORS_ORIGIN=https://yourdomain.com \
  --name proxy-server \
  secure-proxy-server
```

**Using Docker Compose:**
```bash
docker-compose up -d
```

### 2. PM2 Deployment

**Install PM2:**
```bash
npm install -g pm2
```

**Start the server:**
```bash
pm2 start src/index.js --name "proxy-server" -i max
pm2 save
pm2 startup
```

### 3. Vercel Deployment

**Deploy to Vercel:**
```bash
npm install -g vercel
vercel
```

## Security Configuration

### Environment Variables

Required for production:

```env
# Set to production
NODE_ENV=production

# Server port
PORT=3000

# CORS - restrict to your domains
CORS_ORIGIN=https://yourdomain.com,https://app.yourdomain.com

# Rate limiting (adjust based on your needs)
RATE_LIMIT_WINDOW_MS=900000      # 15 minutes
RATE_LIMIT_MAX_REQUESTS=100      # Max requests per window
RATE_LIMIT_PROXY_MAX=50          # Max proxy requests per window
```

### Firewall Rules

Allow only necessary ports:
```bash
# Example using ufw
ufw allow 3000/tcp
ufw enable
```

### Reverse Proxy (Nginx)

Use nginx for SSL/TLS termination:

```nginx
server {
    listen 443 ssl http2;
    server_name proxy.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Security headers (Helmet already sets these, but nginx can too)
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Monitoring

### Health Checks

The server provides a health endpoint:
```bash
curl http://localhost:3000/health
```

### Log Monitoring

**PM2 logs:**
```bash
pm2 logs proxy-server
```

**Docker logs:**
```bash
docker logs -f proxy-server
```

### Performance Monitoring

Monitor rate limits in response headers:
- `RateLimit-Limit`: Maximum requests allowed
- `RateLimit-Remaining`: Requests remaining
- `RateLimit-Reset`: Seconds until limit resets

## Security Best Practices

1. ✅ **Never expose .env files** - They contain sensitive configuration
2. ✅ **Use HTTPS in production** - Configure with reverse proxy
3. ✅ **Restrict CORS origins** - Don't use `*` in production
4. ✅ **Monitor logs regularly** - Watch for suspicious activity
5. ✅ **Keep dependencies updated** - Run `npm audit` regularly
6. ✅ **Use firewall rules** - Limit access to necessary ports
7. ✅ **Rate limit appropriately** - Adjust based on your usage patterns
8. ✅ **Use strong authentication** - Add API keys or OAuth if exposing publicly

## Maintenance

### Update Dependencies

```bash
# Check for vulnerabilities
npm audit

# Fix vulnerabilities
npm audit fix

# Update dependencies
npm update
```

### Backup Configuration

Always backup your:
- Environment variables (.env)
- SSL/TLS certificates
- Application logs
- Configuration files

## Troubleshooting

### Server won't start
- Check if port 3000 is available
- Verify environment variables are set
- Check file permissions
- Review error logs

### Rate limiting too strict
- Adjust `RATE_LIMIT_MAX_REQUESTS` in .env
- Adjust `RATE_LIMIT_PROXY_MAX` for proxy endpoints
- Change `RATE_LIMIT_WINDOW_MS` to modify time window

### SSRF protection blocking valid requests
- Verify the target URL is not localhost or private IP
- Check if URL is properly formatted
- The proxy blocks 10.x.x.x, 172.16-31.x.x, 192.168.x.x by design

## Support

For security issues, please refer to [SECURITY.md](SECURITY.md)

For general issues, please open a GitHub issue.
