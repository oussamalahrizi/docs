# Complete Guide to Spring Boot Application Properties

**Author:** Generated for oussamalahrizi  
**Date:** 2025-07-26  
**Version:** 1.0

## Table of Contents
1. [Overview](#overview)
2. [Property File Formats](#property-file-formats)
3. [Complete YAML Configuration](#complete-yaml-configuration)
4. [Environment Variables & Docker](#environment-variables--docker)
5. [Property Categories Explained](#property-categories-explained)
6. [Docker Setup with Environment Files](#docker-setup-with-environment-files)
7. [Best Practices](#best-practices)

## Overview

Spring Boot application properties allow you to externalize configuration, making your applications environment-agnostic and easily deployable across different environments (development, staging, production).

### Key Benefits:
- **Externalized Configuration**: No need to rebuild for different environments
- **Environment-Specific Settings**: Different configs for dev, test, prod
- **Security**: Sensitive data kept out of source code
- **Flexibility**: Override properties via environment variables, command line, etc.

## Property File Formats

### application.properties
```properties
# Server Configuration
server.port=8080
server.servlet.context-path=/api/v1

# Database
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret

# Redis
spring.redis.host=localhost
spring.redis.port=6379
```

### application.yml (Recommended)
```yaml
server:
  port: 8080
  servlet:
    context-path: /api/v1

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: secret
  redis:
    host: localhost
    port: 6379
```

## Complete YAML Configuration

```yaml
# =============================================================================
# ENTERPRISE SPRING BOOT APPLICATION CONFIGURATION
# =============================================================================

# Application Metadata
app:
  name: ${APP_NAME:Enterprise API}
  version: ${APP_VERSION:1.0.0}
  description: "Production-ready Spring Boot API"

# =============================================================================
# SERVER CONFIGURATION
# =============================================================================
server:
  port: ${SERVER_PORT:8080}
  servlet:
    context-path: ${CONTEXT_PATH:/api/v1}
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,application/json
  tomcat:
    max-threads: ${TOMCAT_MAX_THREADS:200}
    min-spare-threads: ${TOMCAT_MIN_THREADS:20}
    connection-timeout: ${TOMCAT_CONNECTION_TIMEOUT:20000}

# =============================================================================
# SPRING FRAMEWORK CONFIGURATION
# =============================================================================
spring:
  application:
    name: ${SPRING_APP_NAME:enterprise-api}
  
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:development}

  # =============================================================================
  # DATABASE CONFIGURATION
  # =============================================================================
  datasource:
    url: ${DATABASE_URL:jdbc:postgresql://localhost:5432/enterprise}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: ${DB_DRIVER:org.postgresql.Driver}
    
    # HikariCP Connection Pool
    hikari:
      maximum-pool-size: ${DB_POOL_MAX:20}
      minimum-idle: ${DB_POOL_MIN:5}
      connection-timeout: ${DB_CONNECTION_TIMEOUT:30000}
      idle-timeout: ${DB_IDLE_TIMEOUT:600000}
      max-lifetime: ${DB_MAX_LIFETIME:1800000}
      leak-detection-threshold: ${DB_LEAK_DETECTION:60000}

  # JPA/Hibernate Configuration
  jpa:
    hibernate:
      ddl-auto: ${JPA_DDL_AUTO:validate}
    show-sql: ${JPA_SHOW_SQL:false}
    properties:
      hibernate:
        dialect: ${HIBERNATE_DIALECT:org.hibernate.dialect.PostgreSQLDialect}
        format_sql: ${HIBERNATE_FORMAT_SQL:true}
        jdbc:
          batch_size: ${HIBERNATE_BATCH_SIZE:20}

  # =============================================================================
  # REDIS CONFIGURATION
  # =============================================================================
  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    password: ${REDIS_PASSWORD:}
    database: ${REDIS_DATABASE:0}
    timeout: ${REDIS_TIMEOUT:2000ms}
    lettuce:
      pool:
        max-active: ${REDIS_POOL_MAX_ACTIVE:10}
        max-idle: ${REDIS_POOL_MAX_IDLE:10}
        min-idle: ${REDIS_POOL_MIN_IDLE:1}

  # =============================================================================
  # JACKSON JSON CONFIGURATION
  # =============================================================================
  jackson:
    serialization:
      write-dates-as-timestamps: false
      fail-on-empty-beans: false
    deserialization:
      fail-on-unknown-properties: false
    default-property-inclusion: non_null

  # =============================================================================
  # MAIL CONFIGURATION
  # =============================================================================
  mail:
    host: ${MAIL_HOST:smtp.gmail.com}
    port: ${MAIL_PORT:587}
    username: ${MAIL_USERNAME:}
    password: ${MAIL_PASSWORD:}
    properties:
      mail:
        smtp:
          auth: ${MAIL_SMTP_AUTH:true}
          starttls:
            enable: ${MAIL_SMTP_STARTTLS:true}

# =============================================================================
# LOGGING CONFIGURATION
# =============================================================================
logging:
  level:
    root: ${LOG_LEVEL_ROOT:INFO}
    org.springframework.web: ${LOG_LEVEL_SPRING_WEB:INFO}
    org.springframework.security: ${LOG_LEVEL_SECURITY:INFO}
    org.hibernate.SQL: ${LOG_LEVEL_HIBERNATE_SQL:WARN}
    com.enterprise.api: ${LOG_LEVEL_APP:INFO}
  
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  
  file:
    name: ${LOG_FILE_PATH:logs/application.log}
    max-size: ${LOG_FILE_MAX_SIZE:10MB}
    max-history: ${LOG_FILE_MAX_HISTORY:30}

# =============================================================================
# ACTUATOR & MONITORING
# =============================================================================
management:
  endpoints:
    web:
      exposure:
        include: ${ACTUATOR_ENDPOINTS:health,info,metrics}
      base-path: ${ACTUATOR_BASE_PATH:/actuator}
  
  endpoint:
    health:
      show-details: ${ACTUATOR_HEALTH_DETAILS:when-authorized}
  
  metrics:
    export:
      prometheus:
        enabled: ${PROMETHEUS_ENABLED:true}

# =============================================================================
# API DOCUMENTATION (OpenAPI/Swagger)
# =============================================================================
springdoc:
  api-docs:
    path: ${API_DOCS_PATH:/api-docs}
  swagger-ui:
    path: ${SWAGGER_UI_PATH:/swagger-ui.html}
    enabled: ${SWAGGER_ENABLED:true}

# =============================================================================
# SECURITY CONFIGURATION
# =============================================================================
security:
  jwt:
    secret: ${JWT_SECRET:myVerySecretKey123!@#}
    expiration: ${JWT_EXPIRATION:86400000}  # 24 hours
    refresh-expiration: ${JWT_REFRESH_EXPIRATION:604800000}  # 7 days
  
  cors:
    allowed-origins: ${CORS_ALLOWED_ORIGINS:http://localhost:3000,http://localhost:8080}
    allowed-methods: ${CORS_ALLOWED_METHODS:GET,POST,PUT,DELETE,PATCH,OPTIONS}
    allowed-headers: ${CORS_ALLOWED_HEADERS:*}
    allow-credentials: ${CORS_ALLOW_CREDENTIALS:true}
    max-age: ${CORS_MAX_AGE:3600}

# =============================================================================
# RATE LIMITING
# =============================================================================
rate-limiting:
  enabled: ${RATE_LIMITING_ENABLED:true}
  requests-per-minute: ${RATE_LIMIT_REQUESTS:100}
  burst-capacity: ${RATE_LIMIT_BURST:150}

# =============================================================================
# FILE UPLOAD CONFIGURATION
# =============================================================================
file-upload:
  max-file-size: ${MAX_FILE_SIZE:10MB}
  max-request-size: ${MAX_REQUEST_SIZE:50MB}
  upload-directory: ${UPLOAD_DIR:/tmp/uploads}
  allowed-extensions: ${ALLOWED_FILE_EXTENSIONS:jpg,jpeg,png,pdf,docx}

# =============================================================================
# CACHE CONFIGURATION
# =============================================================================
cache:
  redis:
    time-to-live: ${CACHE_TTL:1800}  # 30 minutes
    key-prefix: ${CACHE_KEY_PREFIX:enterprise-api}
  
  caffeine:
    maximum-size: ${CAFFEINE_MAX_SIZE:1000}
    expire-after-write: ${CAFFEINE_EXPIRE_WRITE:300s}

# =============================================================================
# EXTERNAL API CONFIGURATION
# =============================================================================
external-apis:
  payment-service:
    base-url: ${PAYMENT_SERVICE_URL:https://api.payment.com}
    api-key: ${PAYMENT_API_KEY:}
    timeout: ${PAYMENT_TIMEOUT:30s}
  
  notification-service:
    base-url: ${NOTIFICATION_SERVICE_URL:https://api.notifications.com}
    api-key: ${NOTIFICATION_API_KEY:}
    timeout: ${NOTIFICATION_TIMEOUT:15s}

# =============================================================================
# BUSINESS LOGIC CONFIGURATION
# =============================================================================
business:
  pagination:
    default-page-size: ${DEFAULT_PAGE_SIZE:20}
    max-page-size: ${MAX_PAGE_SIZE:100}
  
  validation:
    max-username-length: ${MAX_USERNAME_LENGTH:50}
    min-password-length: ${MIN_PASSWORD_LENGTH:8}
    password-require-special-chars: ${PASSWORD_REQUIRE_SPECIAL:true}
  
  features:
    user-registration: ${FEATURE_USER_REGISTRATION:true}
    email-verification: ${FEATURE_EMAIL_VERIFICATION:true}
    two-factor-auth: ${FEATURE_2FA:false}
    maintenance-mode: ${MAINTENANCE_MODE:false}

# =============================================================================
# PROFILE-SPECIFIC CONFIGURATIONS
# =============================================================================

---
# DEVELOPMENT PROFILE
spring:
  config:
    activate:
      on-profile: development

  # Development Database (H2)
  datasource:
    url: jdbc:h2:mem:devdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  h2:
    console:
      enabled: true
      path: /h2-console
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

# Development Logging
logging:
  level:
    root: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

# Development Features
business:
  features:
    maintenance-mode: false

---
# PRODUCTION PROFILE
spring:
  config:
    activate:
      on-profile: production

# Production Database Pool
  datasource:
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10

# Production Logging
logging:
  level:
    root: WARN
    com.enterprise.api: INFO
    org.springframework.web: WARN

# Production Security
management:
  endpoint:
    health:
      show-details: never

# Production Features
springdoc:
  swagger-ui:
    enabled: false

---
# TESTING PROFILE
spring:
  config:
    activate:
      on-profile: testing

  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 

  jpa:
    hibernate:
      ddl-auto: create-drop

logging:
  level:
    root: WARN
    com.enterprise.api: DEBUG
```

## Environment Variables & Docker

### .env File Structure
Create a `.env` file for your Docker setup:

```bash
# =============================================================================
# DOCKER ENVIRONMENT CONFIGURATION
# =============================================================================

# Application Configuration
APP_NAME=Enterprise API
APP_VERSION=1.0.0
SPRING_PROFILES_ACTIVE=production

# Server Configuration
SERVER_PORT=8080
CONTEXT_PATH=/api/v1

# Database Configuration
DATABASE_URL=jdbc:postgresql://postgres:5432/enterprise
DB_USERNAME=postgres
DB_PASSWORD=your_secure_password_here
DB_POOL_MAX=30
DB_POOL_MIN=5

# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=your_redis_password
REDIS_DATABASE=0

# Security Configuration
JWT_SECRET=your_very_secure_jwt_secret_key_at_least_256_bits_long
JWT_EXPIRATION=86400000
CORS_ALLOWED_ORIGINS=https://yourdomain.com,https://app.yourdomain.com

# Email Configuration
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=noreply@yourdomain.com
MAIL_PASSWORD=your_email_password
MAIL_SMTP_AUTH=true
MAIL_SMTP_STARTTLS=true

# External APIs
PAYMENT_SERVICE_URL=https://api.stripe.com
PAYMENT_API_KEY=sk_live_your_stripe_key
NOTIFICATION_SERVICE_URL=https://api.sendgrid.com
NOTIFICATION_API_KEY=SG.your_sendgrid_key

# Logging Configuration
LOG_LEVEL_ROOT=INFO
LOG_LEVEL_APP=INFO
LOG_FILE_PATH=/app/logs/application.log

# Monitoring
ACTUATOR_ENDPOINTS=health,info,metrics,prometheus
PROMETHEUS_ENABLED=true

# Features
FEATURE_USER_REGISTRATION=true
FEATURE_EMAIL_VERIFICATION=true
FEATURE_2FA=true
MAINTENANCE_MODE=false

# File Upload
MAX_FILE_SIZE=10MB
UPLOAD_DIR=/app/uploads
ALLOWED_FILE_EXTENSIONS=jpg,jpeg,png,pdf,docx

# Rate Limiting
RATE_LIMITING_ENABLED=true
RATE_LIMIT_REQUESTS=1000
RATE_LIMIT_BURST=1500
```

### Docker Compose Configuration

```yaml
version: '3.8'

services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    ports:
      - "${SERVER_PORT:-8080}:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=${SPRING_PROFILES_ACTIVE:-production}
    env_file:
      - .env
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logs:/app/logs
      - ./uploads:/app/uploads
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: enterprise
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Dockerfile Example

```dockerfile
FROM openjdk:17-jdk-slim

WORKDIR /app

# Copy the JAR file
COPY target/enterprise-api-*.jar app.jar

# Create directories for logs and uploads
RUN mkdir -p /app/logs /app/uploads

# Create non-root user
RUN addgroup --system spring && adduser --system spring --ingroup spring
RUN chown -R spring:spring /app
USER spring:spring

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Property Categories Explained

### Server Configuration
| Property | Purpose | Default | Environment Variable |
|----------|---------|---------|---------------------|
| `server.port` | Application port | 8080 | `SERVER_PORT` |
| `server.servlet.context-path` | Base path for APIs | / | `CONTEXT_PATH` |
| `server.tomcat.max-threads` | Max HTTP threads | 200 | `TOMCAT_MAX_THREADS` |

### Database Properties
| Property | Purpose | Environment Variable |
|----------|---------|---------------------|
| `spring.datasource.url` | Database connection URL | `DATABASE_URL` |
| `spring.datasource.username` | Database username | `DB_USERNAME` |
| `spring.datasource.password` | Database password | `DB_PASSWORD` |
| `spring.datasource.hikari.maximum-pool-size` | Max connections | `DB_POOL_MAX` |

### Redis Properties
| Property | Purpose | Environment Variable |
|----------|---------|---------------------|
| `spring.redis.host` | Redis server host | `REDIS_HOST` |
| `spring.redis.port` | Redis server port | `REDIS_PORT` |
| `spring.redis.password` | Redis password | `REDIS_PASSWORD` |

### Security Properties
| Property | Purpose | Environment Variable |
|----------|---------|---------------------|
| `security.jwt.secret` | JWT signing key | `JWT_SECRET` |
| `security.jwt.expiration` | JWT expiration time | `JWT_EXPIRATION` |
| `security.cors.allowed-origins` | CORS allowed origins | `CORS_ALLOWED_ORIGINS` |

### Logging Properties
| Property | Purpose | Environment Variable |
|----------|---------|---------------------|
| `logging.level.root` | Root log level | `LOG_LEVEL_ROOT` |
| `logging.file.name` | Log file location | `LOG_FILE_PATH` |
| `logging.file.max-size` | Max log file size | `LOG_FILE_MAX_SIZE` |

## Docker Setup with Environment Files

### 1. Development Setup
```bash
# Create development environment file
cp .env .env.development

# Edit .env.development for local development
SPRING_PROFILES_ACTIVE=development
DATABASE_URL=jdbc:h2:mem:devdb
LOG_LEVEL_ROOT=DEBUG

# Run with development profile
docker-compose --env-file .env.development up -d
```

### 2. Production Setup
```bash
# Create production environment file
cp .env .env.production

# Edit .env.production for production
SPRING_PROFILES_ACTIVE=production
DATABASE_URL=jdbc:postgresql://prod-postgres:5432/enterprise
LOG_LEVEL_ROOT=WARN
JWT_SECRET=your_super_secure_production_jwt_secret

# Run with production profile
docker-compose --env-file .env.production up -d
```

### 3. Using Docker Secrets (Recommended for Production)
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: your-registry/enterprise-api:latest
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - DATABASE_URL=jdbc:postgresql://postgres:5432/enterprise
    secrets:
      - db_password
      - jwt_secret
      - redis_password
    command: >
      sh -c "
        export DB_PASSWORD=$$(cat /run/secrets/db_password) &&
        export JWT_SECRET=$$(cat /run/secrets/jwt_secret) &&
        export REDIS_PASSWORD=$$(cat /run/secrets/redis_password) &&
        java -jar app.jar
      "

secrets:
  db_password:
    external: true
  jwt_secret:
    external: true
  redis_password:
    external: true
```

## Best Practices

### 1. Environment Variables
- ✅ **Always use environment variables for sensitive data**
- ✅ **Provide default values for non-sensitive properties**
- ✅ **Use descriptive environment variable names**
- ❌ **Never commit secrets to version control**

### 2. Property Organization
- ✅ **Group related properties together**
- ✅ **Use consistent naming conventions**
- ✅ **Document complex properties with comments**
- ✅ **Use profile-specific configurations**

### 3. Security
- ✅ **Use strong, unique secrets for each environment**
- ✅ **Rotate secrets regularly**
- ✅ **Use Docker secrets or external secret management**
- ✅ **Validate input properties**

### 4. Performance
- ✅ **Tune connection pools based on load**
- ✅ **Configure appropriate timeouts**
- ✅ **Set reasonable cache TTL values**
- ✅ **Monitor and adjust based on metrics**

### 5. Deployment
- ✅ **Use different .env files for different environments**
- ✅ **Implement health checks**
- ✅ **Configure proper logging levels**
- ✅ **Test configuration changes in staging first**

---

**Generated on:** 2025-07-26 22:19:11 UTC  
**For:** oussamalahrizi  
**File:** spring-boot-properties-guide.md