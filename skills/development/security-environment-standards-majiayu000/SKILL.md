---
name: security-environment-standards
description: Security and environment configuration standards for web applications, including environment variable management, secure coding practices, and production deployment security. Use when setting up environments, configuring security, or deploying applications.
---

# Security Environment Standards

## Core Security Principles

- **Defense in depth**: Multiple layers of security controls
- **Principle of least privilege**: Minimal access rights for users and systems
- **Secure by default**: Security features enabled by default
- **Regular security updates**: Keep dependencies and systems updated
- **Environment isolation**: Separate development, staging, and production environments

## Environment Configuration

### 1. Environment Variables

```python
# .env file structure (never commit to version control)
# Database Configuration
DATABASE_URL=postgresql://user:password@localhost/dbname
DATABASE_SSL_MODE=require

# API Keys and Secrets
SECRET_KEY=your-secret-key-here
AZURE_COSMOS_KEY=your-cosmos-db-key
OPENAI_API_KEY=your-openai-key
CLAUDE_API_KEY=your-claude-key

# Application Settings
DEBUG=False
FLASK_ENV=production
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com

# External Services
REDIS_URL=redis://localhost:6379
ELASTICSEARCH_URL=https://localhost:9200

# Security Settings
CSRF_SECRET_KEY=csrf-secret-key
SESSION_COOKIE_SECURE=True
SESSION_COOKIE_HTTPONLY=True
```

### 2. Environment-Specific Configuration

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    """Base configuration."""
    SECRET_KEY = os.environ.get('SECRET_KEY')
    if not SECRET_KEY:
        raise ValueError("SECRET_KEY environment variable is required")
    
    DATABASE_URL = os.environ.get('DATABASE_URL')
    REDIS_URL = os.environ.get('REDIS_URL')
    
    # Security headers
    SECURITY_HEADERS = {
        'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
        'X-Content-Type-Options': 'nosniff',
        'X-Frame-Options': 'DENY',
        'X-XSS-Protection': '1; mode=block',
        'Referrer-Policy': 'strict-origin-when-cross-origin'
    }

class DevelopmentConfig(Config):
    """Development configuration."""
    DEBUG = True
    TESTING = False
    DATABASE_URL = os.environ.get('DEV_DATABASE_URL') or 'sqlite:///dev.db'
    
class ProductionConfig(Config):
    """Production configuration."""
    DEBUG = False
    TESTING = False
    
    # Enhanced security for production
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    PERMANENT_SESSION_LIFETIME = 1800  # 30 minutes
    
    # Database connection pooling
    SQLALCHEMY_ENGINE_OPTIONS = {
        'pool_pre_ping': True,
        'pool_recycle': 300,
        'pool_size': 10,
        'max_overflow': 20
    }

class TestingConfig(Config):
    """Testing configuration."""
    TESTING = True
    DATABASE_URL = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False
```

## Security Implementation

### 1. Input Validation and Sanitization

```python
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField
from wtforms.validators import DataRequired, Length, ValidationError
import re
import bleach

class ChuukesesTranslationForm(FlaskForm):
    """Secure form for translation input."""
    
    chuukese_text = TextAreaField(
        'Chuukese Text',
        validators=[
            DataRequired(message="Chuukese text is required"),
            Length(min=1, max=5000, message="Text must be between 1 and 5000 characters")
        ]
    )
    
    def validate_chuukese_text(self, field):
        """Custom validation for Chuukese text."""
        # Allow Chuukese characters, basic punctuation, and whitespace
        allowed_pattern = r'^[a-zA-ZáéíóúāēīōūcfghjklmnprstwxCFGHJKLMNPRSTWX\s.,!?\-\'"\n\r]+$'
        
        if not re.match(allowed_pattern, field.data):
            raise ValidationError("Text contains invalid characters")
        
        # Basic XSS prevention
        cleaned_text = bleach.clean(field.data, tags=[], strip=True)
        if cleaned_text != field.data:
            raise ValidationError("Text contains potentially unsafe content")

def sanitize_user_input(text):
    """Sanitize user input for safe storage and display."""
    if not text:
        return ""
    
    # Remove potentially dangerous HTML/script content
    cleaned = bleach.clean(
        text,
        tags=[],  # No HTML tags allowed
        attributes={},  # No attributes allowed
        strip=True  # Strip disallowed tags
    )
    
    # Additional sanitization for Chuukese text
    # Preserve accented characters while removing other special chars
    safe_pattern = r'[a-zA-ZáéíóúāēīōūcfghjklmnprstwxCFGHJKLMNPRSTWX\s.,!?\-\'"()\n\r]'
    cleaned = ''.join(re.findall(safe_pattern, cleaned))
    
    return cleaned.strip()
```

### 2. Authentication and Authorization

```python
from flask_login import UserMixin, login_required
from werkzeug.security import generate_password_hash, check_password_hash
import secrets
from datetime import datetime, timedelta

class User(UserMixin):
    """Secure user model."""
    
    def __init__(self, username, email):
        self.username = username
        self.email = email
        self.password_hash = None
        self.is_active = True
        self.created_at = datetime.utcnow()
        self.last_login = None
        self.failed_login_attempts = 0
        self.account_locked_until = None
        
    def set_password(self, password):
        """Set password with secure hashing."""
        if len(password) < 8:
            raise ValueError("Password must be at least 8 characters long")
        
        # Check for password complexity
        if not re.search(r'[A-Z]', password):
            raise ValueError("Password must contain at least one uppercase letter")
        if not re.search(r'[a-z]', password):
            raise ValueError("Password must contain at least one lowercase letter")
        if not re.search(r'\d', password):
            raise ValueError("Password must contain at least one digit")
        
        self.password_hash = generate_password_hash(password, method='pbkdf2:sha256', salt_length=32)
    
    def check_password(self, password):
        """Check password with brute force protection."""
        if self.is_account_locked():
            return False
        
        is_correct = check_password_hash(self.password_hash, password)
        
        if is_correct:
            self.failed_login_attempts = 0
            self.last_login = datetime.utcnow()
        else:
            self.failed_login_attempts += 1
            if self.failed_login_attempts >= 5:
                self.account_locked_until = datetime.utcnow() + timedelta(minutes=30)
        
        return is_correct
    
    def is_account_locked(self):
        """Check if account is locked due to failed attempts."""
        if self.account_locked_until and datetime.utcnow() < self.account_locked_until:
            return True
        return False

def require_auth(role=None):
    """Decorator for route authorization."""
    def decorator(f):
        @wraps(f)
        @login_required
        def decorated_function(*args, **kwargs):
            if role and not current_user.has_role(role):
                abort(403)  # Forbidden
            return f(*args, **kwargs)
        return decorated_function
    return decorator
```

### 3. Database Security

```python
from sqlalchemy import text
from sqlalchemy.orm import validates

class SecureDatabaseOperations:
    """Secure database operation patterns."""
    
    @staticmethod
    def secure_query(session, query_template, parameters):
        """Execute parameterized queries to prevent SQL injection."""
        try:
            # Use parameterized queries
            result = session.execute(text(query_template), parameters)
            return result.fetchall()
        except Exception as e:
            # Log security-related errors without exposing details
            logger.error(f"Database security error: {type(e).__name__}")
            raise ValueError("Invalid query parameters")
    
    @staticmethod
    def validate_chuukese_word(word):
        """Validate Chuukese word before database insertion."""
        if not word or len(word.strip()) == 0:
            raise ValueError("Word cannot be empty")
        
        if len(word) > 200:
            raise ValueError("Word exceeds maximum length")
        
        # Check for valid Chuukese characters
        valid_pattern = r'^[a-zA-ZáéíóúāēīōūcfghjklmnprstwxCFGHJKLMNPRSTWX\-\']+$'
        if not re.match(valid_pattern, word.strip()):
            raise ValueError("Word contains invalid characters")
        
        return word.strip()

# Database model with validation
class DictionaryEntry(Base):
    __tablename__ = 'dictionary_entries'
    
    id = Column(Integer, primary_key=True)
    chuukese_word = Column(String(200), nullable=False)
    english_definition = Column(Text, nullable=False)
    
    @validates('chuukese_word')
    def validate_chuukese_word(self, key, word):
        return SecureDatabaseOperations.validate_chuukese_word(word)
    
    @validates('english_definition')
    def validate_definition(self, key, definition):
        if not definition or len(definition.strip()) == 0:
            raise ValueError("Definition cannot be empty")
        
        # Sanitize HTML content
        cleaned_definition = bleach.clean(definition, tags=[], strip=True)
        if len(cleaned_definition) > 5000:
            raise ValueError("Definition exceeds maximum length")
        
        return cleaned_definition
```

### 4. API Security

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
import jwt
from datetime import datetime, timedelta

# Rate limiting
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

class APISecurityManager:
    """API security utilities."""
    
    @staticmethod
    def generate_api_token(user_id, expires_in=3600):
        """Generate secure API token."""
        payload = {
            'user_id': user_id,
            'exp': datetime.utcnow() + timedelta(seconds=expires_in),
            'iat': datetime.utcnow(),
            'jti': secrets.token_urlsafe(32)  # Unique token ID
        }
        return jwt.encode(payload, current_app.config['SECRET_KEY'], algorithm='HS256')
    
    @staticmethod
    def verify_api_token(token):
        """Verify and decode API token."""
        try:
            payload = jwt.decode(
                token,
                current_app.config['SECRET_KEY'],
                algorithms=['HS256'],
                options={'verify_exp': True}
            )
            return payload
        except jwt.ExpiredSignatureError:
            raise ValueError("Token has expired")
        except jwt.InvalidTokenError:
            raise ValueError("Invalid token")

@app.route('/api/translate', methods=['POST'])
@limiter.limit("10 per minute")
def api_translate():
    """Secure API endpoint for translation."""
    # Verify API token
    auth_header = request.headers.get('Authorization')
    if not auth_header or not auth_header.startswith('Bearer '):
        return jsonify({'error': 'Authentication required'}), 401
    
    try:
        token = auth_header.split(' ')[1]
        payload = APISecurityManager.verify_api_token(token)
        user_id = payload['user_id']
    except ValueError as e:
        return jsonify({'error': str(e)}), 401
    
    # Validate input
    data = request.get_json()
    if not data or 'text' not in data:
        return jsonify({'error': 'Text parameter required'}), 400
    
    try:
        # Sanitize input
        text = sanitize_user_input(data['text'])
        if len(text) > 1000:
            return jsonify({'error': 'Text too long'}), 400
        
        # Process translation
        result = translate_text(text)
        return jsonify(result)
        
    except Exception as e:
        logger.error(f"Translation API error: {e}")
        return jsonify({'error': 'Translation failed'}), 500
```

## Production Deployment Security

### 1. HTTPS and SSL/TLS

```nginx
# nginx.conf - Force HTTPS
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;
    
    # SSL Configuration
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;
    
    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 2. Docker Security

```dockerfile
# Dockerfile with security best practices
FROM python:3.11-slim as base

# Create non-root user
RUN adduser --disabled-password --gecos '' --shell /bin/bash appuser

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Change ownership to non-root user
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

# Run application
CMD ["python", "app.py"]
```

## Security Monitoring

### 1. Logging Security Events

```python
import logging
from flask import request
import json

# Configure security logger
security_logger = logging.getLogger('security')
security_handler = logging.FileHandler('security.log')
security_formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
security_handler.setFormatter(security_formatter)
security_logger.addHandler(security_handler)
security_logger.setLevel(logging.INFO)

def log_security_event(event_type, details, severity='INFO'):
    """Log security-related events."""
    event_data = {
        'event_type': event_type,
        'timestamp': datetime.utcnow().isoformat(),
        'ip_address': request.remote_addr,
        'user_agent': request.user_agent.string,
        'details': details
    }
    
    if severity == 'WARNING':
        security_logger.warning(json.dumps(event_data))
    elif severity == 'ERROR':
        security_logger.error(json.dumps(event_data))
    else:
        security_logger.info(json.dumps(event_data))

# Usage examples
@app.before_request
def log_requests():
    """Log suspicious request patterns."""
    # Log requests with suspicious patterns
    suspicious_patterns = [
        'script>',
        'javascript:',
        'onload=',
        'onerror=',
        '../../../',
        'passwd',
        'etc/shadow'
    ]
    
    request_data = str(request.get_data())
    for pattern in suspicious_patterns:
        if pattern in request_data.lower():
            log_security_event(
                'suspicious_request',
                {'pattern': pattern, 'data': request_data[:200]},
                'WARNING'
            )
```

### 2. Security Headers Implementation

```python
from flask import Flask
from flask_talisman import Talisman

app = Flask(__name__)

# Configure security headers
Talisman(app, {
    'strict_transport_security': True,
    'strict_transport_security_max_age': 31536000,
    'content_security_policy': {
        'default-src': "'self'",
        'script-src': ["'self'", "'unsafe-inline'"],
        'style-src': ["'self'", "'unsafe-inline'"],
        'img-src': ["'self'", "data:", "https:"],
        'font-src': ["'self'", "https:"],
        'connect-src': ["'self'"],
        'frame-ancestors': "'none'"
    },
    'referrer_policy': 'strict-origin-when-cross-origin'
})
```

## Best Practices

### 1. Environment Management

- Never commit secrets to version control
- Use separate environments for development, staging, and production
- Rotate API keys and secrets regularly
- Use environment-specific configuration files

### 2. Regular Security Updates

- Keep all dependencies updated
- Monitor security advisories for used packages
- Implement automated security scanning in CI/CD
- Regular security audits and penetration testing

### 3. Data Protection

- Encrypt sensitive data at rest
- Use HTTPS for all communications
- Implement proper session management
- Regular database backups with encryption

### 4. Access Control

- Implement principle of least privilege
- Use strong authentication mechanisms
- Regular access reviews and cleanup
- Proper error handling without information disclosure

## Dependencies

- `flask-limiter`: API rate limiting
- `flask-talisman`: Security headers
- `python-jose`: JWT token handling
- `bleach`: HTML sanitization
- `cryptography`: Encryption utilities

## Validation Criteria

Security implementation should:

- ✅ Use HTTPS in production environments
- ✅ Implement proper input validation and sanitization
- ✅ Use parameterized queries for database operations
- ✅ Include comprehensive logging of security events
- ✅ Implement rate limiting on API endpoints
- ✅ Use secure session and cookie configuration
- ✅ Follow security headers best practices
