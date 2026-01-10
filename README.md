# Native Node.js Proxy Server

Proxy server untuk mengakses HTTP/HTTPS API, download file, dan WebSocket connection melalui proxy. Support SSL certificate bypass dan file besar.

## Quick Start

```bash
# Install dependencies
npm install

# Start server
npm run dev
```

Server berjalan di `http://localhost:3000`

---

## 🔗 HTTP/HTTPS Proxy

**Cara kerja:** Proxy akan meneruskan request Anda ke target URL dan mengembalikan response-nya.

### Basic Usage

**GET Request:**
```bash
curl "http://localhost:3000/proxy?url=https://api.example.com/users"
```

**POST Request dengan JSON body:**
```bash
curl -X POST "http://localhost:3000/proxy?url=https://api.example.com/login" \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"secret123"}'
```

**PUT Request:**
```bash
curl -X PUT "http://localhost:3000/proxy?url=https://api.example.com/users/1" \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'
```

**DELETE Request:**
```bash
curl -X DELETE "http://localhost:3000/proxy?url=https://api.example.com/users/1"
```

**Dengan Headers Kustom:**
```bash
curl "http://localhost:3000/proxy?url=https://api.example.com/data" \
  -H "Authorization: Bearer your-token-here" \
  -H "X-API-Key: your-api-key"
```

### Real Example (API yang Anda pakai)

```bash
# Login ke API
curl -X POST "http://localhost:3000/proxy?url=https://119.13.101.169/api/auth/admin/v1/en/client/auth" \
  -H "Content-Type: application/json" \
  -d '{"id": 1, "secret": "1873hjasdASIqwe97aASawf98th"}'

# Response: {"access_token":"eyJ...","expired_at":"2025-11-13T20:24:17.451794952+08:00","type":"Bearer"}

# Gunakan token untuk GET data
curl "http://localhost:3000/proxy?url=https://119.13.101.169/api/master/admin/v1/en/country/list" \
  -H "Authorization: Bearer eyJ..."
```

---

## 💾 File Download

**Cara kerja:** Download file dengan streaming (memory efficient), support file besar dan resume download.

### Basic Download

```bash
# Download file ke direktori current
curl -O "http://localhost:3000/download?url=https://example.com/largefile.zip"

# Download dengan nama kustom
curl -o my-file.zip "http://localhost:3000/download?url=https://example.com/file.zip"
```

### Resume Download (Lanjutkan Download)

```bash
# Jika download terputus, lanjutkan dengan -C -
curl -C - -O "http://localhost:3000/download?url=https://example.com/largefile.zip"
```

### Download dengan Range Request

```bash
# Download 1MB pertama saja
curl -H "Range: bytes=0-1048575" "http://localhost:3000/download?url=https://example.com/largefile.zip"

# Download dari byte 1MB ke 2MB
curl -H "Range: bytes=1048576-2097151" "http://localhost:3000/download?url=https://example.com/largefile.zip"
```

### Example Download File Besar

```bash
# Download 100MB file (akan menampilkan progress)
curl -O "http://localhost:3000/download?url=https://httpbin.org/bytes/104857600"

# Output di terminal:
# Download progress: 25.50% (2.5 MB/s)
# Download progress: 50.00% (2.8 MB/s)
# Download completed: 100.00 MB in 35.67 seconds
```

---

## 🔌 WebSocket Proxy

**Cara kerja:** Proxy akan membuat jalur WebSocket antara client Anda dan target WebSocket server.

### Cara Pakai WebSocket Proxy

**Step 1:** Dapatkan info WebSocket target
```bash
curl "http://localhost:3000/ws-proxy?url=wss://echo.websocket.org"
```

**Step 2:** Connect ke WebSocket proxy (dengan JavaScript)

```html
<script>
// Connect ke proxy server
const ws = new WebSocket("ws://localhost:3000/ws-proxy");

ws.onopen = function(event) {
  console.log("Connected to proxy");

  // Kirim target WebSocket URL
  ws.send(JSON.stringify({
    target: "wss://echo.websocket.org"
  }));
};

ws.onmessage = function(event) {
  const data = JSON.parse(event.data);

  if (data.type === 'connected') {
    console.log("Connected to target WebSocket:", data.target);
    // Sekarang bisa kirim pesan ke target
    ws.send("Hello from proxy!");
  } else {
    // Pesan dari target WebSocket
    console.log("Received:", data);
  }
};

ws.onerror = function(error) {
  console.error("WebSocket error:", error);
};
</script>
```

### Example: WebSocket Echo Test

```javascript
const ws = new WebSocket("ws://localhost:3000/ws-proxy");

ws.onopen = () => {
  // Connect ke echo server
  ws.send(JSON.stringify({ target: "wss://echo.websocket.org" }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type !== 'connected') {
    console.log("Echo response:", data); // Akan muncul pesan yang sama
  }
};

// Kirim pesan
setTimeout(() => {
  ws.send("Hello WebSocket!");
}, 1000);
```

---

## 🕐 Time & Greeting

**Cara kerja:** Endpoint ini menampilkan waktu server saat ini dengan greeting yang sesuai berdasarkan jam.

### Get Current Time

```bash
curl "http://localhost:3000/time"
```

Response:
```json
{
  "greeting": "Good afternoon",
  "timestamp": "2025-12-24T14:04:21.148Z",
  "utc": "Wed, 24 Dec 2025 14:04:21 GMT",
  "unix": 1766585061,
  "timezone": "UTC",
  "localTime": "12/24/2025, 2:04:21 PM",
  "hour": 14,
  "date": "Wed Dec 24 2025",
  "time": "14:04:21"
}
```

### Greeting Schedule

- **5:00 - 11:59** → "Good morning"
- **12:00 - 16:59** → "Good afternoon"
- **17:00 - 20:59** → "Good evening"
- **21:00 - 4:59** → "Good night"

---

## 🏥 Health Check & Monitoring

### Cek Server Status
```bash
curl "http://localhost:3000/health"
```

Response:
```json
{
  "status": "OK",
  "timestamp": "2025-11-03T12:30:00.000Z",
  "version": "1.0.0",
  "proxyType": "native-nodejs",
  "endpoints": {
    "proxy": "/proxy?url=<target-url>",
    "download": "/download?url=<file-url>",
    "websocket": "/ws-proxy?url=<ws-url>",
    "time": "/time",
    "health": "/health"
  },
  "features": ["HTTP/HTTPS proxy", "File download streaming", "WebSocket proxy"]
}
```

### Lihat Semua Endpoint
```bash
curl "http://localhost:3000/"
```

---

## 🛠️ Usage Examples (Bahasa Pemrograman)

### JavaScript/Node.js

```javascript
// HTTP/HTTPS Proxy
const proxyUrl = `http://localhost:3000/proxy?url=${encodeURIComponent('https://api.example.com/data')}`;

fetch(proxyUrl)
  .then(res => res.json())
  .then(data => console.log(data));

// POST dengan data
const postData = {
  username: "testuser",
  password: "testpass"
};

fetch(`http://localhost:3000/proxy?url=${encodeURIComponent('https://api.example.com/login')}`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(postData)
})
.then(res => res.json())
.then(data => console.log(data));

// WebSocket
const ws = new WebSocket("ws://localhost:3000/ws-proxy");
ws.onopen = () => {
  ws.send(JSON.stringify({ target: "wss://echo.websocket.org" }));
};
ws.onmessage = (event) => console.log(JSON.parse(event.data));

// Get current time with greeting
fetch('http://localhost:3000/time')
  .then(res => res.json())
  .then(data => console.log(`${data.greeting}! Current time: ${data.localTime}`));
```

### Python

```python
import requests
import urllib.parse
import websocket

# HTTP/HTTPS Proxy
api_url = "https://api.example.com/data"
proxy_url = f"http://localhost:3000/proxy?url={urllib.parse.quote(api_url)}"
response = requests.get(proxy_url)
print(response.json())

# POST request
login_data = {"username": "test", "password": "test"}
login_url = "https://api.example.com/login"
proxy_login = f"http://localhost:3000/proxy?url={urllib.parse.quote(login_url)}"
response = requests.post(proxy_login, json=login_data)
print(response.json())

# File download
file_url = "https://example.com/largefile.zip"
download_url = f"http://localhost:3000/download?url={urllib.parse.quote(file_url)}"
response = requests.get(download_url, stream=True)
with open("downloaded_file.zip", "wb") as f:
    for chunk in response.iter_content(chunk_size=8192):
        f.write(chunk)

# Get current time
time_response = requests.get("http://localhost:3000/time")
time_data = time_response.json()
print(f"{time_data['greeting']}! Current time: {time_data['localTime']}")
```

## Configuration

Environment variables di `.env` (copy dari `.env.example`):

```env
PORT=3000
NODE_ENV=production
CORS_ORIGIN=*

# Rate Limiting (requests per 15 minutes)
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_PROXY_MAX=50

# Proxy Configuration
PROXY_TIMEOUT=30000
PROXY_FOLLOW_REDIRECTS=true

# Logging
LOG_LEVEL=info
```

**Important:** Never commit `.env` file to version control. Use `.env.example` as template.

## Security Features

- ✅ **Rate Limiting**: Protects against DoS attacks (100 req/15min general, 50 req/15min for proxy)
- ✅ **SSRF Protection**: Blocks localhost and private IP ranges (10.x, 172.16-31.x, 192.168.x)
- ✅ **Security Headers**: Helmet.js with CSP, HSTS, and other security headers
- ✅ **Input Validation**: URL validation and sanitization
- ✅ **Request Size Limits**: 10MB limit for request body
- ✅ **Error Handling**: Prevents information leakage in production mode
- ✅ **SSL Bypass**: For development (use with caution in production)

## Deployment

### Production Checklist

Before deploying to production:

1. ✅ Set `NODE_ENV=production` in your `.env`
2. ✅ Configure `CORS_ORIGIN` to specific domains (not `*`)
3. ✅ Review and adjust rate limits for your use case
4. ✅ Set up HTTPS/TLS (use reverse proxy like nginx)
5. ✅ Configure firewall rules
6. ✅ Set up monitoring and logging
7. ✅ Never expose `.env` file
8. ✅ Keep dependencies updated (`npm audit`)

### PM2 (Process Manager)
```bash
# Install PM2 globally
npm install -g pm2

# Start the proxy server
pm2 start src/index.js --name "proxy-server"

# View logs
pm2 logs proxy-server

# Monitor
pm2 monit

# Auto-restart on system reboot
pm2 startup
pm2 save
```

### Docker
```dockerfile
FROM node:20-alpine

# Create app directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy source code
COPY src ./src

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership
RUN chown -R nodejs:nodejs /app

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start server
CMD ["npm", "start"]
```

Build and run:
```bash
docker build -t proxy-server .
docker run -d -p 3000:3000 --name proxy-server \
  -e NODE_ENV=production \
  -e CORS_ORIGIN=https://yourdomain.com \
  proxy-server
```

### Vercel (Serverless)

Already configured via `vercel.json`. Deploy with:

```bash
npm install -g vercel
vercel
```

### Nginx Reverse Proxy

For production, use nginx as reverse proxy with SSL:

```nginx
server {
    listen 443 ssl http2;
    server_name proxy.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

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

## Examples

### JavaScript
```javascript
const proxyUrl = `http://localhost:3000/proxy?url=${encodeURIComponent(apiUrl)}`;
fetch(proxyUrl).then(res => res.json()).then(console.log);
```

### Python
```python
import requests, urllib.parse
proxy_url = f'http://localhost:3000/proxy?url={urllib.parse.quote(api_url)}'
response = requests.get(proxyUrl)
```

## Troubleshooting

**Common Issues:**

1. **404 Not Found** - Target URL doesn't exist → Check URL directly
2. **Connection timeout** - Server not reachable → Check network/firewall
3. **SSL Certificate Error** - SSL expired → Proxy bypasses automatically (use with caution)
4. **Invalid URL format** - Wrong URL format → Use proper URL encoding
5. **429 Too Many Requests** - Rate limit exceeded → Wait 15 minutes or adjust limits
6. **SSRF Protection Error** - Trying to access private IPs → This is blocked for security

**Debug Tips:**
- Check server logs for detailed errors
- Test target URL directly before proxying
- Use browser developer tools
- Verify environment variables are set correctly
- Check rate limit headers in response

**Rate Limit Headers:**
- `RateLimit-Limit`: Maximum requests allowed
- `RateLimit-Remaining`: Requests remaining in window
- `RateLimit-Reset`: Time when limit resets

## Monitoring & Logging

Monitor your proxy server:

```bash
# View logs with PM2
pm2 logs proxy-server

# View Docker logs
docker logs -f proxy-server

# Check server health
curl http://localhost:3000/health
```

## Security Considerations

⚠️ **Important Security Notes:**

1. **Private Network Access**: The proxy blocks access to private IP ranges by default (localhost, 10.x.x.x, 172.16-31.x.x, 192.168.x.x)
2. **SSL Certificate Bypass**: The proxy bypasses SSL certificate validation - use only in trusted environments
3. **Public Exposure**: Do NOT expose this proxy directly to the public internet without additional security measures
4. **Authentication**: Consider adding authentication/API keys for production use
5. **Allowed Domains**: Use domain whitelisting in production environments
6. **Regular Updates**: Run `npm audit` regularly and keep dependencies updated

## License

MIT License

## Project Structure

```
proxy/
├── src/
│   └── index.js              # Main proxy server
├── package.json              # Dependencies
├── README.md                 # Documentation
├── .env                      # Environment variables
└── .env.example              # Environment template
```