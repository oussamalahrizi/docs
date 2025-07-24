# Project Structure Comparison

This section compares Django and Spring Boot project organization, helping you understand how to structure your Spring Boot application based on your Django experience.

## Django Project Structure

### Typical Django Project Layout
```
myproject/
├── manage.py                    # Django CLI tool
├── requirements.txt             # Python dependencies
├── myproject/                   # Main project package
│   ├── __init__.py
│   ├── settings.py             # Configuration
│   ├── urls.py                 # Main URL routing
│   ├── wsgi.py                 # WSGI server config
│   └── asgi.py                 # ASGI server config
├── users/                      # User management app
│   ├── __init__.py
│   ├── models.py               # Data models
│   ├── views.py                # Business logic/views
│   ├── urls.py                 # App-specific URLs
│   ├── serializers.py          # DRF serializers
│   ├── admin.py                # Admin interface
│   ├── apps.py                 # App configuration
│   └── migrations/             # Database migrations
├── blog/                       # Blog app
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── migrations/
├── static/                     # Static files
├── media/                      # User uploads
└── templates/                  # HTML templates
```

## Spring Boot Project Structure

### Equivalent Spring Boot Layout
```
myproject/
├── pom.xml                     # Maven dependencies (or build.gradle)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/company/myproject/
│   │   │       ├── MyProjectApplication.java    # Main application class
│   │   │       ├── config/                      # Configuration classes
│   │   │       │   ├── DatabaseConfig.java
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── WebConfig.java
│   │   │       ├── user/                        # User module
│   │   │       │   ├── model/
│   │   │       │   │   └── User.java           # Entity (Django model)
│   │   │       │   ├── repository/
│   │   │       │   │   └── UserRepository.java # Data access
│   │   │       │   ├── service/
│   │   │       │   │   ├── UserService.java    # Business logic
│   │   │       │   │   └── impl/
│   │   │       │   │       └── UserServiceImpl.java
│   │   │       │   ├── controller/
│   │   │       │   │   └── UserController.java # REST endpoints
│   │   │       │   └── dto/
│   │   │       │       ├── UserDTO.java        # Data transfer objects
│   │   │       │       └── CreateUserRequest.java
│   │   │       ├── blog/                        # Blog module
│   │   │       │   ├── model/
│   │   │       │   ├── repository/
│   │   │       │   ├── service/
│   │   │       │   ├── controller/
│   │   │       │   └── dto/
│   │   │       ├── common/                      # Shared components
│   │   │       │   ├── exception/
│   │   │       │   │   ├── GlobalExceptionHandler.java
│   │   │       │   │   └── CustomExceptions.java
│   │   │       │   ├── util/
│   │   │       │   └── constant/
│   │   │       └── security/                    # Security components
│   │   │           ├── JwtAuthenticationFilter.java
│   │   │           └── CustomUserDetailsService.java
│   │   └── resources/
│   │       ├── application.yml                  # Configuration (Django settings.py)
│   │       ├── application-dev.yml              # Development config
│   │       ├── application-prod.yml             # Production config
│   │       ├── static/                          # Static web resources
│   │       ├── templates/                       # Thymeleaf templates (if used)
│   │       └── db/migration/                    # Flyway migrations
│   │           ├── V1__Create_user_table.sql
│   │           └── V2__Create_blog_table.sql
│   └── test/
│       └── java/
│           └── com/company/myproject/
│               ├── user/
│               │   ├── UserServiceTest.java
│               │   └── UserControllerTest.java
│               └── integration/
│                   └── UserIntegrationTest.java
├── target/                     # Build output (Maven)
└── docker/                     # Docker configuration
    ├── Dockerfile
    └── docker-compose.yml
```

## Key Structural Differences

### Package Organization vs App Organization

#### Django Apps
- **Purpose**: Logical grouping of related functionality
- **Structure**: Each app is a Python package with predefined files
- **Registration**: Apps must be registered in `INSTALLED_APPS`

```python
# Django app structure
users/
├── models.py       # All user-related models
├── views.py        # All user-related views
├── urls.py         # URL routing for this app
└── serializers.py  # API serializers
```

#### Spring Boot Modules
- **Purpose**: Package-based organization following domain-driven design
- **Structure**: Java packages with layer-based organization
- **Registration**: Automatic component scanning

```java
// Spring Boot module structure
user/
├── model/          # Entities (similar to Django models)
├── repository/     # Data access layer
├── service/        # Business logic layer
├── controller/     # Web/API layer
└── dto/           # Data transfer objects
```

### Layer Separation

#### Django Approach
```python
# models.py - Data models
class User(models.Model):
    username = models.CharField(max_length=150)
    email = models.EmailField()

# views.py - Business logic and web layer
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

# serializers.py - Data serialization
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'
```

#### Spring Boot Approach
```java
// model/User.java - Entity
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String email;
}

// repository/UserRepository.java - Data access
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

// service/UserService.java - Business logic
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User createUser(CreateUserRequest request) {
        // Business logic here
    }
}

// controller/UserController.java - Web layer
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@RequestBody CreateUserRequest request) {
        // Controller logic here
    }
}

// dto/UserDTO.java - Data transfer
public class UserDTO {
    private Long id;
    private String username;
    private String email;
}
```

## Configuration Files

### Django Configuration
```python
# settings.py
DEBUG = True
SECRET_KEY = 'your-secret-key'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myproject',
        'USER': 'user',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'rest_framework',
    'users',
    'blog',
]
```

### Spring Boot Configuration
```yaml
# application.yml
spring:
  application:
    name: myproject
  
  datasource:
    url: jdbc:postgresql://localhost:5432/myproject
    username: user
    password: password
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

server:
  port: 8080

logging:
  level:
    com.company.myproject: DEBUG
```

## Migration Strategy

### From Django Apps to Spring Boot Modules

1. **Identify Django Apps**: List all your Django apps
2. **Map to Domains**: Group related apps into business domains
3. **Create Package Structure**: Create Java packages for each domain
4. **Implement Layers**: Separate concerns into model, repository, service, controller

#### Example Migration

**Django Structure:**
```
users/
├── models.py
├── views.py
└── serializers.py

profiles/
├── models.py
├── views.py
└── serializers.py
```

**Spring Boot Structure:**
```
user/  # Combined users + profiles
├── model/
│   ├── User.java
│   └── UserProfile.java
├── repository/
├── service/
├── controller/
└── dto/
```

## Build and Dependency Management

### Django
```python
# requirements.txt
Django==4.2.0
djangorestframework==3.14.0
psycopg2-binary==2.9.5
celery==5.2.0
```

### Spring Boot (Maven)
```xml
<!-- pom.xml -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
    </dependency>
</dependencies>
```

## Development Tools

### Django Development
```bash
# Django management commands
python manage.py runserver
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py shell
```

### Spring Boot Development
```bash
# Maven commands
mvn spring-boot:run
mvn clean install
mvn test

# Gradle commands (alternative)
./gradlew bootRun
./gradlew build
./gradlew test
```

## Best Practices

### Django App Design
- Keep apps focused and cohesive
- Avoid circular dependencies between apps
- Use clear, descriptive app names
- Separate concerns within each app

### Spring Boot Module Design
- Follow domain-driven design principles
- Use clear package naming conventions
- Implement proper layer separation
- Leverage dependency injection for loose coupling

## Summary

| Aspect | Django | Spring Boot |
|--------|--------|-------------|
| **Organization** | App-based | Package-based |
| **Configuration** | Python settings | YAML/Properties + Java config |
| **Dependencies** | requirements.txt | pom.xml/build.gradle |
| **Entry Point** | manage.py | Application.java |
| **Layer Separation** | File-based | Package-based |
| **Auto-discovery** | INSTALLED_APPS | Component scanning |

The key transition is moving from Django's file-based organization to Spring Boot's package and layer-based organization, while maintaining the same logical separation of concerns.