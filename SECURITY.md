# Security Policy

## Supported Versions

The following versions of this proxy server are currently being supported with security updates:

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take the security of our proxy server seriously. If you discover a security vulnerability, please follow these steps:

### How to Report

1. **DO NOT** open a public issue on GitHub
2. Email the security team at: [Your security contact email - replace this]
3. Include the following information:
   - Type of vulnerability
   - Full paths of source file(s) related to the vulnerability
   - Location of the affected source code (tag/branch/commit or direct URL)
   - Step-by-step instructions to reproduce the issue
   - Proof-of-concept or exploit code (if possible)
   - Impact of the vulnerability

### Response Timeline

- **Initial Response**: Within 48 hours of report
- **Status Updates**: Weekly updates on the progress
- **Fix Timeline**: We aim to release a fix within 30 days for critical vulnerabilities

### What to Expect

- **Accepted Vulnerabilities**: We will work on a fix and coordinate a release date with you
- **Declined Reports**: We will explain why we don't consider it a vulnerability
- **Public Disclosure**: We follow responsible disclosure practices and will credit you (unless you prefer to remain anonymous)

## Security Best Practices

When deploying this proxy server:

1. **Environment Variables**: Never commit `.env` files to version control
2. **HTTPS**: Always use HTTPS in production
3. **Rate Limiting**: Keep rate limiting enabled (configured by default)
4. **Access Control**: Restrict access to the proxy using firewall rules or authentication
5. **Monitoring**: Monitor logs for suspicious activity
6. **Updates**: Keep dependencies up to date with `npm audit` and `npm update`
7. **Private Networks**: Be cautious when deploying on networks with access to internal resources

## Security Features

This proxy server includes:

- Rate limiting (100 requests per 15 minutes per IP)
- Helmet.js security headers
- Input validation and sanitization
- Protection against SSRF (Server-Side Request Forgery) attacks
- Request size limits
- CORS configuration
- SSL/TLS support

## Known Limitations

- The proxy bypasses SSL certificate validation by design (for development purposes)
- Private IP ranges are blocked by default to prevent SSRF attacks
- This is intended for development and testing, not for public production use without additional security measures
