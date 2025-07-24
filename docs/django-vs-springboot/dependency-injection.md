# Apps vs Modules and Dependency Injection

This section explains how Django's app-based architecture translates to Spring Boot's module structure and how Django's implicit dependencies become explicit dependency injection in Spring Boot.

## Django Apps Architecture

### Django App Concept
In Django, apps are reusable Python packages that encapsulate related functionality:

```python
# users/apps.py
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'users'
    
    def ready(self):
        # Import signals when app is ready
        import users.signals

# users/models.py
from django.db import models

class User(models.Model):
    username = models.CharField(max_length=150, unique=True)
    email = models.EmailField(unique=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)

# users/views.py
from django.shortcuts import get_object_or_404
from rest_framework import viewsets, status
from rest_framework.response import Response
from .models import User
from .serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
    def create(self, request):
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            user = serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# users/services.py (if you separate business logic)
from django.core.mail import send_mail
from .models import User

class UserService:
    @staticmethod
    def create_user_with_welcome_email(user_data):
        user = User.objects.create(**user_data)
        UserService.send_welcome_email(user)
        return user
    
    @staticmethod
    def send_welcome_email(user):
        send_mail(
            'Welcome!',
            f'Welcome {user.username}!',
            'from@example.com',
            [user.email],
        )

# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'rest_framework',
    'users',  # App registration
    'blog',
    'notifications',
]
```

### Django Inter-App Dependencies
```python
# blog/models.py
from django.db import models
from users.models import User  # Direct import

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

# blog/views.py
from users.models import User
from notifications.services import NotificationService  # Direct import

class PostViewSet(viewsets.ModelViewSet):
    def create(self, request):
        # Business logic mixed with view logic
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            post = serializer.save()
            
            # Direct service call
            NotificationService.notify_followers(post.author, post)
            
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

## Spring Boot Module Architecture

### Spring Boot Module Structure
Spring Boot uses package-based modules with explicit dependency injection:

```java
// Application main class
@SpringBootApplication
@EnableJpaRepositories
@ComponentScan(basePackages = "com.company.myproject")
public class MyProjectApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyProjectApplication.class, args);
    }
}

// user/model/User.java - Entity (Django Model equivalent)
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String username;
    
    @Column(unique = true)
    private String email;
    
    @Column(name = "is_active")
    private Boolean isActive = true;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Post> posts = new ArrayList<>();
}

// user/repository/UserRepository.java - Data Access Layer
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    List<User> findByIsActiveTrue();
    
    @Query("SELECT u FROM User u WHERE u.username LIKE %:query% OR u.email LIKE %:query%")
    List<User> searchUsers(@Param("query") String query);
}

// user/service/UserService.java - Business Logic Layer
@Service
@Transactional
@Slf4j
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;  // Injected dependency
    private final ModelMapper modelMapper;
    
    // Constructor injection (preferred)
    public UserService(UserRepository userRepository, 
                      EmailService emailService, 
                      ModelMapper modelMapper) {
        this.userRepository = userRepository;
        this.emailService = emailService;
        this.modelMapper = modelMapper;
    }
    
    public UserDTO createUser(CreateUserRequest request) {
        // Validation
        if (userRepository.findByEmail(request.getEmail()).isPresent()) {
            throw new EmailAlreadyExistsException("Email already exists: " + request.getEmail());
        }
        
        // Create user
        User user = User.builder()
                .username(request.getUsername())
                .email(request.getEmail())
                .isActive(true)
                .build();
        
        User savedUser = userRepository.save(user);
        
        // Send welcome email (delegated to injected service)
        emailService.sendWelcomeEmail(savedUser);
        
        // Convert to DTO
        return modelMapper.map(savedUser, UserDTO.class);
    }
    
    public Optional<UserDTO> findByEmail(String email) {
        return userRepository.findByEmail(email)
                .map(user -> modelMapper.map(user, UserDTO.class));
    }
    
    public List<UserDTO> searchUsers(String query) {
        return userRepository.searchUsers(query).stream()
                .map(user -> modelMapper.map(user, UserDTO.class))
                .collect(Collectors.toList());
    }
}

// user/controller/UserController.java - Web Layer
@RestController
@RequestMapping("/api/v1/users")
@CrossOrigin(origins = "*")
@Validated
@Slf4j
public class UserController {
    
    private final UserService userService;  // Injected dependency
    
    // Constructor injection
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        log.info("Creating user with email: {}", request.getEmail());
        
        UserDTO user = userService.createUser(request);
        
        return ResponseEntity.status(HttpStatus.CREATED).body(user);
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<UserDTO>> searchUsers(@RequestParam String query) {
        List<UserDTO> users = userService.searchUsers(query);
        return ResponseEntity.ok(users);
    }
}
```

### Cross-Module Dependencies with Dependency Injection

```java
// blog/model/Post.java
@Entity
@Table(name = "posts")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @Column(columnDefinition = "TEXT")
    private String content;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    private User author;  // Reference to User entity
    
    @CreationTimestamp
    private LocalDateTime createdAt;
}

// blog/service/PostService.java
@Service
@Transactional
@Slf4j
public class PostService {
    
    private final PostRepository postRepository;
    private final UserService userService;  // Cross-module dependency
    private final NotificationService notificationService;  // Another dependency
    private final ModelMapper modelMapper;
    
    // All dependencies injected through constructor
    public PostService(PostRepository postRepository,
                      UserService userService,
                      NotificationService notificationService,
                      ModelMapper modelMapper) {
        this.postRepository = postRepository;
        this.userService = userService;
        this.notificationService = notificationService;
        this.modelMapper = modelMapper;
    }
    
    public PostDTO createPost(CreatePostRequest request, String authorEmail) {
        // Get user through injected service
        UserDTO author = userService.findByEmail(authorEmail)
                .orElseThrow(() -> new UserNotFoundException("User not found: " + authorEmail));
        
        // Create post
        Post post = Post.builder()
                .title(request.getTitle())
                .content(request.getContent())
                .author(modelMapper.map(author, User.class))
                .build();
        
        Post savedPost = postRepository.save(post);
        
        // Notify followers through injected service
        notificationService.notifyFollowers(author, savedPost);
        
        return modelMapper.map(savedPost, PostDTO.class);
    }
}

// notification/service/NotificationService.java
@Service
@Async  // Asynchronous processing
@Slf4j
public class NotificationService {
    
    private final NotificationRepository notificationRepository;
    private final EmailService emailService;
    private final UserService userService;
    
    public NotificationService(NotificationRepository notificationRepository,
                             EmailService emailService,
                             UserService userService) {
        this.notificationRepository = notificationRepository;
        this.emailService = emailService;
        this.userService = userService;
    }
    
    @Async
    public void notifyFollowers(UserDTO author, Post post) {
        log.info("Notifying followers about new post: {}", post.getTitle());
        
        // Get followers (this would typically involve a follower relationship)
        List<UserDTO> followers = getFollowers(author.getId());
        
        followers.forEach(follower -> {
            // Create notification record
            Notification notification = Notification.builder()
                    .recipientId(follower.getId())
                    .message(String.format("%s posted: %s", author.getUsername(), post.getTitle()))
                    .type(NotificationType.NEW_POST)
                    .isRead(false)
                    .build();
            
            notificationRepository.save(notification);
            
            // Send email notification
            emailService.sendNotificationEmail(follower.getEmail(), notification.getMessage());
        });
    }
    
    private List<UserDTO> getFollowers(Long userId) {
        // Implementation would depend on your follower model
        return Collections.emptyList();
    }
}

// common/service/EmailService.java
@Service
@Slf4j
public class EmailService {
    
    private final JavaMailSender mailSender;
    private final String fromEmail;
    
    public EmailService(JavaMailSender mailSender, 
                       @Value("${app.email.from}") String fromEmail) {
        this.mailSender = mailSender;
        this.fromEmail = fromEmail;
    }
    
    @Async
    public void sendWelcomeEmail(User user) {
        try {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(fromEmail);
            message.setTo(user.getEmail());
            message.setSubject("Welcome!");
            message.setText(String.format("Welcome %s!", user.getUsername()));
            
            mailSender.send(message);
            log.info("Welcome email sent to: {}", user.getEmail());
        } catch (Exception e) {
            log.error("Failed to send welcome email to: {}", user.getEmail(), e);
        }
    }
    
    @Async
    public void sendNotificationEmail(String email, String message) {
        try {
            SimpleMailMessage mailMessage = new SimpleMailMessage();
            mailMessage.setFrom(fromEmail);
            mailMessage.setTo(email);
            mailMessage.setSubject("New Notification");
            mailMessage.setText(message);
            
            mailSender.send(mailMessage);
            log.info("Notification email sent to: {}", email);
        } catch (Exception e) {
            log.error("Failed to send notification email to: {}", email, e);
        }
    }
}
```

## Dependency Injection Types

### Constructor Injection (Recommended)
```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor injection - immutable dependencies
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
}
```

### Field Injection (Not Recommended)
```java
@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;  // Field injection
    
    @Autowired
    private EmailService emailService;
    
    // Problems: harder to test, mutable dependencies, reflection required
}
```

### Setter Injection (Rare Use Cases)
```java
@Service
public class UserService {
    
    private UserRepository userRepository;
    private EmailService emailService;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

## Configuration and Bean Definition

### Java Configuration
```java
@Configuration
@ComponentScan(basePackages = "com.company.myproject")
public class ApplicationConfig {
    
    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration()
                .setMatchingStrategy(MatchingStrategies.STRICT)
                .setFieldMatchingEnabled(true);
        return mapper;
    }
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    @Bean
    @Primary
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
    
    // Conditional bean creation
    @Bean
    @ConditionalOnProperty(name = "app.features.analytics", havingValue = "true")
    public AnalyticsService analyticsService() {
        return new AnalyticsServiceImpl();
    }
}
```

### Profile-Based Configuration
```java
@Configuration
@Profile("development")
public class DevConfig {
    
    @Bean
    @Primary
    public EmailService emailService() {
        return new MockEmailService();  // Mock for development
    }
}

@Configuration
@Profile("production")
public class ProdConfig {
    
    @Bean
    @Primary
    public EmailService emailService(JavaMailSender mailSender, 
                                   @Value("${app.email.from}") String fromEmail) {
        return new SmtpEmailService(mailSender, fromEmail);
    }
}
```

## Testing with Dependency Injection

### Unit Testing
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @Mock
    private ModelMapper modelMapper;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void createUser_ShouldCreateUserAndSendEmail() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
                .username("testuser")
                .email("test@example.com")
                .build();
        
        User savedUser = User.builder()
                .id(1L)
                .username("testuser")
                .email("test@example.com")
                .build();
        
        UserDTO expectedDto = UserDTO.builder()
                .id(1L)
                .username("testuser")
                .email("test@example.com")
                .build();
        
        when(userRepository.findByEmail(request.getEmail())).thenReturn(Optional.empty());
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        when(modelMapper.map(savedUser, UserDTO.class)).thenReturn(expectedDto);
        
        // When
        UserDTO result = userService.createUser(request);
        
        // Then
        assertThat(result).isEqualTo(expectedDto);
        verify(emailService).sendWelcomeEmail(savedUser);
        verify(userRepository).save(any(User.class));
    }
}
```

### Integration Testing
```java
@SpringBootTest
@TestPropertySource(locations = "classpath:application-test.properties")
@Transactional
class UserServiceIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void createUser_IntegrationTest() {
        // Given
        CreateUserRequest request = CreateUserRequest.builder()
                .username("integrationtest")
                .email("integration@test.com")
                .build();
        
        // When
        UserDTO result = userService.createUser(request);
        
        // Then
        assertThat(result.getUsername()).isEqualTo("integrationtest");
        assertThat(result.getEmail()).isEqualTo("integration@test.com");
        
        Optional<User> savedUser = userRepository.findByEmail("integration@test.com");
        assertThat(savedUser).isPresent();
    }
}
```

## Migration Strategy: From Django Apps to Spring Boot Modules

### Step 1: Identify Django App Boundaries
```python
# Current Django structure
users/          # User management
├── models.py
├── views.py
├── serializers.py
└── services.py

blog/           # Blog functionality
├── models.py
├── views.py
├── serializers.py
└── services.py

notifications/  # Notification system
├── models.py
├── views.py
├── serializers.py
└── services.py
```

### Step 2: Map to Spring Boot Modules
```java
// Target Spring Boot structure
user/
├── model/
├── repository/
├── service/
├── controller/
└── dto/

blog/
├── model/
├── repository/
├── service/
├── controller/
└── dto/

notification/
├── model/
├── repository/
├── service/
├── controller/
└── dto/
```

### Step 3: Identify Dependencies
```python
# Django: Implicit dependencies (direct imports)
from users.models import User
from notifications.services import NotificationService
```

```java
// Spring Boot: Explicit dependencies (constructor injection)
public PostService(UserService userService, NotificationService notificationService) {
    this.userService = userService;
    this.notificationService = notificationService;
}
```

### Step 4: Convert Services
```python
# Django service (static methods, direct imports)
class UserService:
    @staticmethod
    def create_user_with_welcome_email(user_data):
        user = User.objects.create(**user_data)
        send_mail(...)  # Direct mail sending
        return user
```

```java
// Spring Boot service (dependency injection)
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public UserDTO createUser(CreateUserRequest request) {
        User user = userRepository.save(...);
        emailService.sendWelcomeEmail(user);  // Injected service
        return convertToDto(user);
    }
}
```

## Best Practices

### Django App Best Practices
- Keep apps focused on single responsibility
- Avoid circular imports between apps
- Use services to encapsulate business logic
- Minimize direct model imports across apps

### Spring Boot Module Best Practices
- Use constructor injection over field injection
- Keep dependencies minimal and focused
- Use interfaces for service contracts
- Leverage Spring profiles for environment-specific beans
- Write tests that mock dependencies properly
- Use `@Qualifier` for multiple implementations

## Summary

| Aspect | Django Apps | Spring Boot Modules |
|--------|-------------|-------------------|
| **Organization** | Python packages | Java packages |
| **Registration** | INSTALLED_APPS | Component scanning |
| **Dependencies** | Direct imports | Dependency injection |
| **Coupling** | Tight (direct imports) | Loose (interface-based) |
| **Testing** | Manual mocking | Framework-supported mocking |
| **Configuration** | settings.py | @Configuration classes |
| **Lifecycle** | AppConfig.ready() | @PostConstruct, @PreDestroy |

The key transition is moving from Django's implicit dependencies through direct imports to Spring Boot's explicit dependency injection, which provides better testability, modularity, and inversion of control.