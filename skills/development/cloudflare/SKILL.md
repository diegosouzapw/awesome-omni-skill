---
name: cloudflare
description: Use when configuring Cloudflare platform services (DNS, SSL/TLS, WAF, security) or deploying static sites (Pages). For Workers/serverless functions, use the cloudflare-workers skill instead.
version: 2.0.0
license: MIT
---

# Cloudflare Platform Configuration

Comprehensive platform configuration guide for Cloudflare services. For serverless Workers development, see the [cloudflare-workers](../cloudflare-workers/SKILL.md) skill.

## Overview

Cloudflare provides a complete edge platform with:
- Global CDN and DNS
- SSL/TLS management
- Web Application Firewall (WAF)
- DDoS protection
- Static site hosting (Pages)
- Edge computing (Workers - see separate skill)

**Core principle**: This skill covers platform configuration and setup. For Workers development (serverless functions, D1, KV, R2, Durable Objects, AI), use the dedicated cloudflare-workers skill.

## When to Use This Skill

Use this skill when:
- Configuring DNS records and domain settings
- Setting up SSL/TLS certificates
- Configuring firewall rules and security settings
- Deploying static sites to Cloudflare Pages
- Managing WAF rules and bot protection
- Setting up redirects and page rules

**DO NOT** use this skill for:
- Workers development → use [cloudflare-workers](../cloudflare-workers/SKILL.md)
- D1, KV, R2, Durable Objects → use [cloudflare-workers](../cloudflare-workers/SKILL.md)
- AI features (Workers AI) → use [cloudflare-workers](../cloudflare-workers/SKILL.md)

## Service Selection Guide

| Need | Service | When to Use | Reference |
|------|---------|-------------|-----------|
| **DNS Management** | Cloudflare DNS | Domain nameserver configuration, DNS records | [DNS Guide](references/dns.md) |
| **SSL/TLS Certificates** | SSL/TLS | HTTPS encryption, certificate management | [SSL Guide](references/ssl.md) |
| **Security Rules** | WAF + Firewall | Block threats, rate limiting, bot protection | [Firewall Guide](references/firewall.md) |
| **Static Sites** | Pages | Frontend deployment (React, Vue, etc.) | [Pages Guide](references/pages.md) |
| **Serverless Functions** | Workers | API backends, edge compute | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |
| **Databases** | D1 | SQL databases on edge | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |
| **Key-Value Storage** | KV | Cache, sessions, config | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |
| **Object Storage** | R2 | File/media storage | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |
| **Real-time Apps** | Durable Objects | WebSockets, coordination | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |
| **AI/ML** | Workers AI | Edge AI inference | [cloudflare-workers skill](../cloudflare-workers/SKILL.md) |

## Quick Start

### Prerequisites

```bash
# Install Wrangler CLI (needed for Pages)
npm install -g wrangler

# Login to Cloudflare
wrangler login
```

### Initial Setup Workflow

1. **Add Domain to Cloudflare**
   - Log in to Cloudflare Dashboard
   - Click "Add a Site"
   - Enter your domain name
   - Select plan (Free/Pro/Business/Enterprise)
   - Update nameservers at your registrar

2. **Configure DNS**
   - Set up DNS records for your domain
   - See [DNS Configuration Guide](references/dns.md)

3. **Enable SSL/TLS**
   - Configure SSL/TLS mode
   - Set up certificates
   - See [SSL/TLS Guide](references/ssl.md)

4. **Set Up Security**
   - Configure firewall rules
   - Enable WAF protection
   - See [Firewall Guide](references/firewall.md)

5. **Deploy Application**
   - **Static Sites**: Use Pages → [Pages Guide](references/pages.md)
   - **Dynamic/API**: Use Workers → [cloudflare-workers skill](../cloudflare-workers/SKILL.md)

## Common Configuration Tasks

### Verify Domain Setup

```bash
# Check nameservers
dig NS yourdomain.com

# Check DNS propagation
dig A yourdomain.com
dig CNAME www.yourdomain.com
```

### Quick DNS Configuration

**Common Record Types**:
- **A**: IPv4 address (e.g., `example.com` → `192.0.2.1`)
- **AAAA**: IPv6 address
- **CNAME**: Alias (e.g., `www` → `example.com`)
- **MX**: Email routing
- **TXT**: Verification and SPF/DKIM

See [DNS Guide](references/dns.md) for detailed configuration.

### Enable SSL/TLS (Quick)

**Recommended Settings**:
1. SSL/TLS Mode: "Full (strict)" for best security
2. Enable "Always Use HTTPS"
3. Enable "Automatic HTTPS Rewrites"
4. Set minimum TLS version to 1.2+

See [SSL Guide](references/ssl.md) for certificate details.

### Basic Security Rules

**Common Patterns**:
- Block by country: `(ip.geoip.country eq "XX")`
- Rate limiting: Limit requests per IP/path
- Bot protection: Challenge suspicious traffic
- Allow/block IPs: Whitelist/blacklist specific IPs

See [Firewall Guide](references/firewall.md) for rule syntax.

## Platform Architecture Patterns

### Static Site with API Backend

```
┌─────────────────────────────────┐
│  Cloudflare Pages (Frontend)    │
│  - React/Vue/Svelte/etc.        │
│  - Static assets                │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  Workers (API Layer)            │
│  - Business logic               │
│  - Database access (D1)         │
│  - Authentication               │
└─────────────────────────────────┘
```

**Setup**:
1. Deploy frontend to Pages: [Pages Guide](references/pages.md)
2. Deploy API to Workers: [cloudflare-workers skill](../cloudflare-workers/SKILL.md)
3. Configure CORS and custom domains

### Full-Stack Application

```
┌─────────────────────────────────┐
│  Pages Functions (Full-Stack)   │
│  - Frontend + API in one         │
│  - Directory-based routing      │
└─────────────────────────────────┘
```

**Setup**:
1. Use Pages with Functions: [Pages Guide](references/pages.md)
2. Add `functions/` directory for API routes
3. Deploy via Git or CLI

### Multi-Domain Setup

```
example.com        → Pages (marketing site)
app.example.com    → Pages (web app)
api.example.com    → Workers (API)
cdn.example.com    → R2 (media storage)
```

**Setup**:
1. Configure DNS records: [DNS Guide](references/dns.md)
2. Set up SSL certificates: [SSL Guide](references/ssl.md)
3. Deploy each service separately

## Cloudflare Dashboard Navigation

**Key Sections**:
- **Overview**: Traffic stats, security events
- **DNS**: Manage DNS records
- **SSL/TLS**: Certificate management
- **Firewall**: Security rules, WAF
- **Speed**: Caching, optimization
- **Workers & Pages**: Serverless deployments
- **Analytics**: Traffic insights

## Environment Management

### Multiple Environments Pattern

```toml
# Development
DOMAIN = "dev.example.com"
API_URL = "https://api-dev.example.com"

# Staging
DOMAIN = "staging.example.com"
API_URL = "https://api-staging.example.com"

# Production
DOMAIN = "example.com"
API_URL = "https://api.example.com"
```

**Best Practices**:
- Use separate Cloudflare zones for production
- Use subdomains for staging/dev
- Test SSL/DNS changes in non-production first
- Use different API keys per environment

## Monitoring and Analytics

### Key Metrics to Monitor

**DNS**:
- Query volume
- Response times
- DNSSEC status

**SSL/TLS**:
- Certificate expiration
- TLS version usage
- Cipher suite distribution

**Security**:
- Threat detection rate
- Firewall rule triggers
- Bot traffic percentage

**Pages/Workers**:
- Request volume
- Error rates
- Response times

### Setting Up Alerts

Configure alerts for:
- High error rates (5xx responses)
- SSL certificate expiration (30 days before)
- DDoS attack detection
- Unusual traffic spikes

## Cost Optimization

### Free Plan Features
- Unlimited DNS queries
- Free SSL certificates
- DDoS protection
- 100,000 Workers requests/day
- 500 Pages builds/month

### Paid Plan Benefits
- Advanced WAF rules
- Image optimization
- Load balancing
- Priority support
- Increased rate limits

**Cost-Saving Tips**:
1. Use R2 for storage (zero egress fees)
2. Leverage edge caching to reduce origin requests
3. Use Pages for static sites (no compute costs)
4. Monitor usage in dashboard

## Integration with Other Services

### Custom Domains
1. Add domain to Cloudflare
2. Update nameservers at registrar
3. Configure DNS records
4. Set up SSL certificates

### GitHub Integration (Pages)
- Automatic deployments on push
- Preview deployments for PRs
- Build configurations per branch

### CI/CD Integration
```yaml
# GitHub Actions example
- name: Deploy to Cloudflare Pages
  uses: cloudflare/pages-action@v1
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    projectName: my-project
```

## Troubleshooting Common Issues

### DNS Not Resolving
- **Symptom**: Domain not loading
- **Check**: Nameservers updated at registrar (24-48hr propagation)
- **Solution**: Verify with `dig NS yourdomain.com`

### SSL Certificate Errors
- **Symptom**: Browser shows "Not Secure" or certificate warning
- **Check**: SSL mode in dashboard, origin certificate
- **Solution**: Use "Full (strict)" mode with valid origin cert

### Pages Build Failures
- **Symptom**: Deployment fails in build step
- **Check**: Build command, output directory, environment variables
- **Solution**: Check build logs, verify configuration

### Rate Limiting Issues
- **Symptom**: 429 errors or blocked requests
- **Check**: Firewall rules, rate limiting settings
- **Solution**: Adjust thresholds or whitelist IPs

### CORS Errors
- **Symptom**: Browser blocks cross-origin requests
- **Check**: CORS headers in Worker/Pages Functions
- **Solution**: Add proper `Access-Control-Allow-Origin` headers

## Security Best Practices

### DNS Security
- Enable DNSSEC for domain integrity
- Use CAA records to control certificate issuance
- Implement DMARC/SPF/DKIM for email

### SSL/TLS Security
- Use "Full (strict)" mode
- Enable "Always Use HTTPS"
- Set minimum TLS version to 1.2 or higher
- Enable "Opportunistic Encryption"

### Firewall Security
- Block known malicious IPs/countries
- Implement rate limiting on login/API endpoints
- Use Challenge pages for suspicious traffic
- Enable Bot Fight Mode

### Access Control
- Use API tokens with minimal permissions
- Rotate API keys regularly
- Enable 2FA on Cloudflare account
- Audit user permissions

## Detailed Configuration References

For in-depth guides on specific topics:

- **[DNS Configuration](references/dns.md)** - Complete DNS setup, record types, DNSSEC
- **[SSL/TLS Setup](references/ssl.md)** - Certificate management, TLS modes, edge certificates
- **[Firewall & WAF](references/firewall.md)** - Security rules, rate limiting, bot protection
- **[Cloudflare Pages](references/pages.md)** - Static site deployment, Pages Functions
- **[Workers Development](../cloudflare-workers/SKILL.md)** - Serverless functions, D1, KV, R2, AI

## Additional Resources

- **Official Docs**: https://developers.cloudflare.com
- **Dashboard**: https://dash.cloudflare.com
- **Status Page**: https://www.cloudflarestatus.com
- **Community**: https://community.cloudflare.com
- **Discord**: https://discord.cloudflare.com

## Configuration Checklist

### Initial Platform Setup
- [ ] Add domain to Cloudflare
- [ ] Update nameservers at registrar
- [ ] Verify DNS propagation (24-48 hours)
- [ ] Configure DNS records for services
- [ ] Enable SSL/TLS (Full strict mode)
- [ ] Set up HTTPS redirects

### Security Configuration
- [ ] Enable DNSSEC
- [ ] Configure firewall rules
- [ ] Enable WAF (if on paid plan)
- [ ] Set up rate limiting
- [ ] Enable bot protection
- [ ] Configure security headers

### Deployment Setup
- [ ] Choose deployment method (Pages/Workers)
- [ ] Configure build settings (if using Pages)
- [ ] Set environment variables
- [ ] Test in staging environment
- [ ] Deploy to production
- [ ] Verify deployment

### Monitoring Setup
- [ ] Configure analytics
- [ ] Set up alerts for critical events
- [ ] Monitor SSL certificate expiration
- [ ] Review security logs regularly
- [ ] Track performance metrics
