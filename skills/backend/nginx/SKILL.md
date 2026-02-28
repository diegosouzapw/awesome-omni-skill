---
name: nginx
description: "Comprehensive Nginx configuration guide covering server blocks, location matching, reverse proxy, SSL/TLS, load balancing, caching, compression, rate limiting, security headers, access control, logging, HTTP/2 and HTTP/3, rewrites, try_files, API gateway patterns, and performance tuning. Use when configuring Nginx as a web server, reverse proxy, load balancer, or API gateway."
version: 1.0.0
---

# Nginx

## 1. Philosophy

Nginx is an **event-driven, non-blocking web server** that excels at serving static content, reverse proxying, load balancing, and acting as an API gateway. Its configuration is declarative -- you describe the desired behavior, and Nginx handles the event loop.

**Key principles**:
- Configuration is hierarchical. Directives inherit from outer blocks to inner blocks.
- Location matching has specific precedence rules. Know them or debug endlessly.
- Upstream blocks handle backend pools. Never hardcode backend addresses in location blocks.
- Security headers are not optional. Every response must include them.
- Test before reload. Always run `nginx -t` before applying configuration changes.

---

## 2. Server Blocks (Virtual Hosts)

### Basic Server Block

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    root /var/www/example.com/public;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Multiple Domains

```nginx
# Catch-all for undefined domains
server {
    listen 80 default_server;
    server_name _;
    return 444;  # Close connection without response
}

# Application domain
server {
    listen 80;
    server_name app.example.com;
    # ...
}

# API domain
server {
    listen 80;
    server_name api.example.com;
    # ...
}
```

---

## 3. Location Matching

Nginx processes location blocks in a specific precedence order. Understanding this order prevents subtle routing bugs.

### Matching Precedence (Highest to Lowest)

```nginx
# 1. Exact match (highest priority)
location = /health {
    return 200 "OK";
    add_header Content-Type text/plain;
}

# 2. Preferential prefix match (stops searching)
location ^~ /static/ {
    root /var/www;
    expires 30d;
}

# 3. Regular expression match (first match wins, case-sensitive)
location ~ \.php$ {
    fastcgi_pass unix:/run/php/php-fpm.sock;
}

# 4. Regular expression match (case-insensitive)
location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
    expires 365d;
    add_header Cache-Control "public, immutable";
}

# 5. Prefix match (longest match wins)
location /api/ {
    proxy_pass http://backend;
}

# 6. Default prefix match
location / {
    try_files $uri $uri/ /index.html;
}
```

### Common Mistake

```nginx
# BAD: regex location takes precedence over prefix
location /api/ {
    proxy_pass http://backend;
}

location ~ ^/api/internal {
    return 403;  # This OVERRIDES the /api/ prefix above
}

# GOOD: Use ^~ to prevent regex override
location ^~ /api/internal {
    return 403;
}

location /api/ {
    proxy_pass http://backend;
}
```

---

## 4. Reverse Proxy

### Basic Proxy

```nginx
upstream backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://backend;

        # Pass original client information
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 5s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;

        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }
}
```

### WebSocket Proxy

```nginx
location /ws {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;

    # Prevent WebSocket timeout
    proxy_read_timeout 86400s;
    proxy_send_timeout 86400s;
}
```

### Trailing Slash Behavior

```nginx
# With trailing slash: /api/ -> http://backend/
# Strips the matched prefix
location /api/ {
    proxy_pass http://backend/;
}
# Request: /api/users -> proxied as /users

# Without trailing slash: /api/ -> http://backend/api/
# Preserves the full URI
location /api/ {
    proxy_pass http://backend;
}
# Request: /api/users -> proxied as /api/users
```

---

## 5. SSL/TLS

### Modern TLS Configuration

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;

    # Certificates
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # Protocol versions -- TLS 1.2 and 1.3 only
    ssl_protocols TLSv1.2 TLSv1.3;

    # Cipher suites
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # OCSP Stapling -- faster TLS handshake
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    resolver 1.1.1.1 8.8.8.8 valid=300s;
    resolver_timeout 5s;

    # Session caching
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # HSTS -- force HTTPS for 2 years
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
}
```

### Let's Encrypt with Certbot

```nginx
# HTTP challenge location
server {
    listen 80;
    server_name example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}
```

```bash
# Obtain certificate
certbot certonly --webroot -w /var/www/certbot -d example.com -d www.example.com

# Auto-renewal (certbot installs a timer/cron by default)
certbot renew --dry-run

# Reload nginx after renewal
certbot renew --deploy-hook "systemctl reload nginx"
```

---

## 6. Load Balancing

### Upstream Configuration

```nginx
upstream api_servers {
    # Round-robin (default)
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
    server 10.0.0.3:3000;

    # Backup server -- only used when all others are down
    server 10.0.0.4:3000 backup;

    # Mark a server as permanently down
    server 10.0.0.5:3000 down;

    # Keepalive connections to backends
    keepalive 32;
}
```

### Load Balancing Algorithms

```nginx
# Least connections -- send to the server with fewest active connections
upstream api_servers {
    least_conn;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
}

# IP hash -- same client always goes to the same server (sticky sessions)
upstream api_servers {
    ip_hash;
    server 10.0.0.1:3000;
    server 10.0.0.2:3000;
}

# Weighted -- distribute based on server capacity
upstream api_servers {
    server 10.0.0.1:3000 weight=3;  # Gets 3x the traffic
    server 10.0.0.2:3000 weight=1;
}
```

### Health Checks (Nginx Plus or OpenResty)

```nginx
# Passive health checks (open-source Nginx)
upstream api_servers {
    server 10.0.0.1:3000 max_fails=3 fail_timeout=30s;
    server 10.0.0.2:3000 max_fails=3 fail_timeout=30s;
}
```

---

## 7. Caching

### Proxy Cache

```nginx
# Define cache zone in http block
proxy_cache_path /var/cache/nginx/api
    levels=1:2
    keys_zone=api_cache:10m     # 10MB for keys (approx 80,000 keys)
    max_size=1g                 # 1GB on disk
    inactive=60m                # Remove items unused for 60 minutes
    use_temp_path=off;

server {
    location /api/ {
        proxy_pass http://backend;
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;      # Cache 200 responses for 10 minutes
        proxy_cache_valid 404 1m;       # Cache 404 responses for 1 minute
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503;
        proxy_cache_lock on;            # Only one request populates the cache
        proxy_cache_key "$scheme$request_method$host$request_uri";

        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

### Static File Caching

```nginx
location /static/ {
    root /var/www;
    expires 365d;
    add_header Cache-Control "public, immutable";
    access_log off;
}

# Hashed filenames (e.g., app.abc123.js) -- cache forever
location ~* \.[a-f0-9]{8,}\.(js|css|png|jpg|svg|woff2)$ {
    root /var/www;
    expires max;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

---

## 8. Compression

```nginx
# Enable gzip in http block
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 5;          # 1-9, balance between CPU and compression ratio
gzip_min_length 256;        # Do not compress tiny responses
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    application/rss+xml
    image/svg+xml;

# Brotli (requires ngx_brotli module)
brotli on;
brotli_comp_level 6;
brotli_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    image/svg+xml;
```

---

## 9. Rate Limiting

```nginx
# Define rate limit zones in http block
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login_limit:10m rate=1r/s;

server {
    # API rate limiting -- allow burst of 20 with no delay for first 10
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        limit_req_status 429;
        proxy_pass http://backend;
    }

    # Strict rate limiting on login endpoint
    location /api/auth/login {
        limit_req zone=login_limit burst=5;
        limit_req_status 429;
        proxy_pass http://backend;
    }
}
```

### Connection Limiting

```nginx
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

server {
    location /downloads/ {
        limit_conn conn_limit 5;     # Max 5 concurrent connections per IP
        limit_rate 500k;             # Limit bandwidth to 500KB/s per connection
    }
}
```

---

## 10. Security Headers

```nginx
# Add to every server block (or http block for global)
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "0" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://api.example.com; frame-ancestors 'none'; base-uri 'self'; form-action 'self';" always;

# HSTS (only on HTTPS server blocks)
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

---

## 11. Access Control

```nginx
# IP-based access control
location /admin/ {
    allow 10.0.0.0/8;
    allow 192.168.1.0/24;
    deny all;
    proxy_pass http://admin_backend;
}

# Basic authentication
location /staging/ {
    auth_basic "Staging Environment";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://staging_backend;
}

# Block specific user agents
if ($http_user_agent ~* (bot|crawler|spider)) {
    return 403;
}

# Block access to hidden files
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}
```

```bash
# Create htpasswd file
# Install: apt install apache2-utils
htpasswd -c /etc/nginx/.htpasswd admin
```

---

## 12. Logging

### Custom Log Formats

```nginx
# Define in http block
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent"';

log_format json escape=json '{'
    '"time": "$time_iso8601",'
    '"remote_addr": "$remote_addr",'
    '"request_method": "$request_method",'
    '"request_uri": "$request_uri",'
    '"status": $status,'
    '"body_bytes_sent": $body_bytes_sent,'
    '"request_time": $request_time,'
    '"upstream_response_time": "$upstream_response_time",'
    '"http_user_agent": "$http_user_agent",'
    '"http_referer": "$http_referer"'
'}';

access_log /var/log/nginx/access.log json;
error_log /var/log/nginx/error.log warn;
```

### Conditional Logging

```nginx
# Do not log health checks
map $request_uri $loggable {
    ~*^/health  0;
    ~*^/ready   0;
    default     1;
}

access_log /var/log/nginx/access.log main if=$loggable;

# Do not log static file access
location /static/ {
    access_log off;
}
```

---

## 13. HTTP/2 and HTTP/3

```nginx
# HTTP/2 (widely supported)
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    # ...

    # HTTP/2 push (deprecated in most browsers, avoid)
    # Use 103 Early Hints instead
    location / {
        add_header Link "</style.css>; rel=preload; as=style";
        proxy_pass http://backend;
    }
}

# HTTP/3 (QUIC -- requires Nginx 1.25+ with quic module)
server {
    listen 443 ssl;
    listen 443 quic reuseport;

    http2 on;
    http3 on;

    # Advertise HTTP/3 support
    add_header Alt-Svc 'h3=":443"; ma=86400' always;

    ssl_certificate     /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;

    # Required for QUIC
    ssl_early_data on;
}
```

---

## 14. Rewrites and Redirects

```nginx
# Permanent redirect (301)
location /old-page {
    return 301 /new-page;
}

# Redirect with query string preservation
location /search {
    return 301 /find$is_args$args;
}

# Rewrite (internal -- URL changes but client does not see it)
rewrite ^/blog/(\d{4})/(\d{2})/(.+)$ /posts?year=$1&month=$2&slug=$3 last;

# Rewrite with redirect
rewrite ^/legacy/(.*)$ /modern/$1 permanent;  # 301
rewrite ^/temp/(.*)$ /current/$1 redirect;     # 302

# Canonical domain redirect
server {
    listen 80;
    server_name www.example.com;
    return 301 https://example.com$request_uri;
}
```

---

## 15. try_files

```nginx
# Single Page Application (SPA) -- serve index.html for all routes
location / {
    root /var/www/app;
    try_files $uri $uri/ /index.html;
}

# Static files first, then proxy to backend
location / {
    root /var/www/public;
    try_files $uri @backend;
}

location @backend {
    proxy_pass http://app_server;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}

# Custom 404 page
location / {
    root /var/www;
    try_files $uri $uri/ =404;
    error_page 404 /404.html;
}
```

---

## 16. Nginx as API Gateway

```nginx
# Rate limiting, authentication, and routing in one place
upstream auth_service {
    server 127.0.0.1:4000;
    keepalive 16;
}

upstream user_service {
    server 127.0.0.1:4001;
    keepalive 16;
}

upstream order_service {
    server 127.0.0.1:4002;
    keepalive 16;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # Global rate limit
    limit_req zone=api_limit burst=50 nodelay;

    # CORS headers
    add_header Access-Control-Allow-Origin "https://app.example.com" always;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Authorization, Content-Type" always;

    # Preflight requests
    if ($request_method = OPTIONS) {
        add_header Access-Control-Allow-Origin "https://app.example.com";
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Authorization, Content-Type";
        add_header Access-Control-Max-Age 86400;
        return 204;
    }

    # Route to microservices
    location /api/v1/auth/ {
        proxy_pass http://auth_service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/v1/users/ {
        proxy_pass http://user_service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /api/v1/orders/ {
        proxy_pass http://order_service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Request size limit for file uploads
    location /api/v1/uploads/ {
        client_max_body_size 50m;
        proxy_pass http://user_service/uploads/;
    }
}
```

---

## 17. Performance Tuning

```nginx
# nginx.conf -- main context
worker_processes auto;          # One worker per CPU core
worker_rlimit_nofile 65535;     # Max open files per worker

events {
    worker_connections 4096;    # Max connections per worker
    multi_accept on;            # Accept multiple connections at once
    use epoll;                  # Linux optimal event method
}

http {
    # Sendfile -- bypass user space for static files
    sendfile on;
    tcp_nopush on;              # Send headers and file in one packet
    tcp_nodelay on;             # Disable Nagle's algorithm for small packets

    # Timeouts
    keepalive_timeout 65;
    keepalive_requests 1000;
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;

    # Buffers
    client_body_buffer_size 16k;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 4 8k;

    # File descriptor cache
    open_file_cache max=1000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;

    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    access_log /var/log/nginx/access.log main buffer=16k flush=5s;
    error_log /var/log/nginx/error.log warn;
}
```

---

## 18. Anti-Patterns

### NEVER

- Edit configuration without running `nginx -t` before reloading
- Use `if` blocks in location context for anything other than `return` or `rewrite` -- it causes subtle bugs
- Hardcode backend addresses in location blocks -- use upstream blocks
- Serve static files through a proxy when Nginx can serve them directly
- Disable access logging in production -- you need it for debugging
- Use `ssl on` (deprecated) -- use `listen 443 ssl` instead
- Run Nginx as root for worker processes -- only the master process needs root
- Skip security headers -- add them in every server block
- Use self-signed certificates in production -- use Let's Encrypt
- Ignore the `always` parameter on `add_header` -- without it, headers are not added to error responses

### ALWAYS

- Run `nginx -t` before `nginx -s reload`
- Use `upstream` blocks for backend servers
- Enable gzip/brotli compression for text-based content
- Set appropriate `client_max_body_size` for each endpoint
- Use `try_files` for SPA routing instead of nested if/rewrite blocks
- Set `keepalive` connections to upstream backends
- Include security headers on every response
- Log in structured JSON format for production
- Configure SSL with modern cipher suites and TLS 1.2+
- Monitor error logs and upstream response times

---

## 19. Quick Reference

```bash
# Test configuration
nginx -t

# Reload configuration (graceful -- no downtime)
nginx -s reload

# Stop (graceful)
nginx -s quit

# Stop (immediate)
nginx -s stop

# Show compiled modules
nginx -V

# Show active configuration
nginx -T

# Check which process is listening
ss -tlnp | grep :80
```
