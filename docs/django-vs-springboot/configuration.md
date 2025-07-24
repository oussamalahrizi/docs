# Configuration Management

This section covers how Django settings translate to Spring Boot configuration, including environment management, external configuration, and best practices.

## Django Settings Overview

### Basic Django Settings Structure
```python
# settings.py
import os
from pathlib import Path

# Build paths inside the project
BASE_DIR = Path(__file__).resolve().parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get('SECRET_KEY', 'django-insecure-default-key')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.environ.get('DEBUG', 'False').lower() == 'true'

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'myproject'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD', 'password'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'users',
    'blog',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# REST Framework configuration
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Celery configuration
CELERY_BROKER_URL = os.environ.get('REDIS_URL', 'redis://localhost:6379/0')
CELERY_RESULT_BACKEND = os.environ.get('REDIS_URL', 'redis://localhost:6379/0')

# Cache configuration
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL', 'redis://127.0.0.1:6379/1'),
    }
}

# Email configuration
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = os.environ.get('EMAIL_HOST', 'smtp.gmail.com')
EMAIL_PORT = int(os.environ.get('EMAIL_PORT', '587'))
EMAIL_USE_TLS = True
EMAIL_HOST_USER = os.environ.get('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = os.environ.get('EMAIL_HOST_PASSWORD')

# Logging configuration
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'django.log',
            'formatter': 'verbose',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'INFO',
    },
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'level': 'INFO',
            'propagate': False,
        },
    },
}
```

## Spring Boot Configuration Equivalent

### application.yml (Main Configuration)
```yaml
# application.yml
spring:
  application:
    name: myproject
  
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:dev}
  
  datasource:
    url: jdbc:postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME:myproject}
    username: ${DB_USER:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
  
  jpa:
    hibernate:
      ddl-auto: ${DDL_AUTO:validate}
    show-sql: ${SHOW_SQL:false}
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        use_sql_comments: true
  
  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      database: ${REDIS_DB:0}
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
  
  cache:
    type: redis
    redis:
      time-to-live: 600000
  
  mail:
    host: ${EMAIL_HOST:smtp.gmail.com}
    port: ${EMAIL_PORT:587}
    username: ${EMAIL_HOST_USER}
    password: ${EMAIL_HOST_PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
  
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB

server:
  port: ${SERVER_PORT:8080}
  servlet:
    context-path: ${CONTEXT_PATH:}
  error:
    include-message: always
    include-binding-errors: always

# Security configuration
security:
  jwt:
    secret: ${JWT_SECRET:mySecretKey}
    expiration: ${JWT_EXPIRATION:86400} # 24 hours in seconds

# Application-specific configuration
app:
  cors:
    allowed-origins: ${CORS_ALLOWED_ORIGINS:http://localhost:3000,http://localhost:8080}
    allowed-methods: ${CORS_ALLOWED_METHODS:GET,POST,PUT,DELETE,OPTIONS}
    allowed-headers: ${CORS_ALLOWED_HEADERS:*}
  
  pagination:
    default-page-size: ${DEFAULT_PAGE_SIZE:20}
    max-page-size: ${MAX_PAGE_SIZE:100}
  
  file-upload:
    max-size: ${MAX_FILE_SIZE:10MB}
    allowed-types: ${ALLOWED_FILE_TYPES:jpg,jpeg,png,pdf,doc,docx}
    upload-dir: ${UPLOAD_DIR:./uploads}

# Logging configuration
logging:
  level:
    com.company.myproject: ${LOG_LEVEL:DEBUG}
    org.springframework.security: ${SECURITY_LOG_LEVEL:DEBUG}
    org.springframework.web: ${WEB_LOG_LEVEL:INFO}
    org.hibernate.SQL: ${SQL_LOG_LEVEL:DEBUG}
    org.hibernate.type.descriptor.sql.BasicBinder: ${SQL_PARAM_LOG_LEVEL:TRACE}
  
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  
  file:
    name: ${LOG_FILE:./logs/application.log}
    max-size: 10MB
    max-history: 30

# Management endpoints
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  
  info:
    env:
      enabled: true
```

### Environment-Specific Configuration

#### Development Configuration
```yaml
# application-dev.yml
spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  
  h2:
    console:
      enabled: true

server:
  port: 8080

logging:
  level:
    com.company.myproject: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG

app:
  cors:
    allowed-origins: "http://localhost:3000,http://localhost:8080"
```

#### Production Configuration
```yaml
# application-prod.yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

server:
  port: 8080

logging:
  level:
    com.company.myproject: INFO
    org.springframework.web: WARN
    org.hibernate.SQL: WARN
  
  file:
    name: /var/log/myproject/application.log

app:
  cors:
    allowed-origins: "https://myapp.com,https://www.myapp.com"
```

#### Test Configuration
```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password: password
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
  
  h2:
    console:
      enabled: true

logging:
  level:
    com.company.myproject: DEBUG
```

## Java Configuration Classes

### Main Configuration Class
```java
@Configuration
@EnableConfigurationProperties({AppProperties.class, SecurityProperties.class})
@EnableCaching
@EnableAsync
public class ApplicationConfig {
    
    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration()
                .setMatchingStrategy(MatchingStrategies.STRICT)
                .setFieldMatchingEnabled(true)
                .setFieldAccessLevel(org.modelmapper.config.Configuration.AccessLevel.PRIVATE);
        return mapper;
    }
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    @Primary
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
}
```

### Database Configuration
```java
@Configuration
@EnableJpaRepositories(basePackages = "com.company.myproject.*.repository")
@EnableTransactionManagement
public class DatabaseConfig {
    
    @Bean
    @ConfigurationProperties("spring.datasource")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }
    
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory);
        return transactionManager;
    }
    
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
        hibernateJpaVendorAdapter.setShowSql(false);
        hibernateJpaVendorAdapter.setGenerateDdl(true);
        hibernateJpaVendorAdapter.setDatabase(Database.POSTGRESQL);
        return hibernateJpaVendorAdapter;
    }
}
```

### CORS Configuration
```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {
    
    @Value("${app.cors.allowed-origins}")
    private String[] allowedOrigins;
    
    @Value("${app.cors.allowed-methods}")
    private String[] allowedMethods;
    
    @Value("${app.cors.allowed-headers}")
    private String[] allowedHeaders;
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins(allowedOrigins)
                .allowedMethods(allowedMethods)
                .allowedHeaders(allowedHeaders)
                .allowCredentials(true);
    }
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/uploads/**")
                .addResourceLocations("file:./uploads/");
    }
}
```

## Configuration Properties Classes

### Application Properties
```java
@ConfigurationProperties(prefix = "app")
@Data
@Component
public class AppProperties {
    
    private Cors cors = new Cors();
    private Pagination pagination = new Pagination();
    private FileUpload fileUpload = new FileUpload();
    
    @Data
    public static class Cors {
        private String[] allowedOrigins;
        private String[] allowedMethods;
        private String[] allowedHeaders;
    }
    
    @Data
    public static class Pagination {
        private int defaultPageSize = 20;
        private int maxPageSize = 100;
    }
    
    @Data
    public static class FileUpload {
        private String maxSize = "10MB";
        private String[] allowedTypes;
        private String uploadDir = "./uploads";
    }
}
```

### Security Properties
```java
@ConfigurationProperties(prefix = "security")
@Data
@Component
public class SecurityProperties {
    
    private Jwt jwt = new Jwt();
    
    @Data
    public static class Jwt {
        private String secret;
        private long expiration = 86400; // 24 hours
    }
}
```

## Environment Variables and Profiles

### Docker Environment
```dockerfile
# Dockerfile
FROM openjdk:17-jdk-slim

ENV SPRING_PROFILES_ACTIVE=prod
ENV DB_HOST=postgres
ENV DB_NAME=myproject
ENV DB_USER=postgres
ENV DB_PASSWORD=password
ENV REDIS_HOST=redis

COPY target/myproject-*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      DB_HOST: postgres
      DB_NAME: myproject
      DB_USER: postgres
      DB_PASSWORD: password
      REDIS_HOST: redis
      JWT_SECRET: mySecretProductionKey
    depends_on:
      - postgres
      - redis
  
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: myproject
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:6-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Configuration Validation

### Custom Configuration Validator
```java
@Component
@ConfigurationPropertiesBinding
public class ConfigurationValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return AppProperties.class.equals(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        AppProperties properties = (AppProperties) target;
        
        if (properties.getPagination().getDefaultPageSize() <= 0) {
            errors.rejectValue("pagination.defaultPageSize", 
                "invalid.pagination.defaultPageSize", 
                "Default page size must be positive");
        }
        
        if (properties.getPagination().getMaxPageSize() < properties.getPagination().getDefaultPageSize()) {
            errors.rejectValue("pagination.maxPageSize", 
                "invalid.pagination.maxPageSize", 
                "Max page size must be greater than default page size");
        }
    }
}
```

## Migration Strategy

### From Django Settings to Spring Boot Configuration

1. **Identify Configuration Categories**:
   - Database settings → `spring.datasource`
   - Debug/logging → `logging` configuration
   - Static files → `spring.web.resources`
   - Middleware → Filters/Interceptors configuration

2. **Environment Variable Mapping**:
   ```python
   # Django
   DEBUG = os.environ.get('DEBUG', 'False').lower() == 'true'
   ```
   ```yaml
   # Spring Boot
   logging:
     level:
       com.company.myproject: ${LOG_LEVEL:INFO}
   ```

3. **Configuration Organization**:
   - Move environment-specific settings to profile-specific YAML files
   - Create `@ConfigurationProperties` classes for complex configurations
   - Use `@Value` annotations for simple property injection

### Best Practices

#### Django Settings Best Practices
- Use environment variables for sensitive data
- Separate settings by environment (dev, staging, prod)
- Keep settings files in version control (except secrets)
- Use django-environ for better environment variable handling

#### Spring Boot Configuration Best Practices
- Use YAML for hierarchical configuration
- Leverage Spring profiles for environment-specific configs
- Create typed configuration properties classes
- Use `@ConfigurationProperties` for complex configurations
- Validate configuration properties at startup
- Document configuration options clearly

## Summary
```Markdown
| Aspect | Django | Spring Boot |
|--------|--------|-------------|
| **Main Config** | settings.py | application.yml |
| **Environment Vars** | os.environ.get() | ${VAR:default} |
| **Environment-specific** | Multiple settings files | Profile-specific YAML |
| **Validation** | Manual | @ConfigurationProperties validation |
| **Type Safety** | Limited | Strong typing with classes |
| **Auto-completion** | Limited | Full IDE support |
| **Documentation** | Comments | @ConfigurationProperties metadata |
```

Spring Boot provides more structured, type-safe configuration management compared to Django's Python-based approach, with better IDE integration and validation capabilities.