# Models and ORM Comparison

This section compares Django's model system and ORM with Spring Boot's JPA entities and Spring Data, showing how to translate your Django models to JPA entities.

## Django Models Overview

### Basic Django Model
```python
# users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import MinLengthValidator

class User(AbstractUser):
    email = models.EmailField(unique=True)
    phone_number = models.CharField(max_length=20, blank=True, null=True)
    date_of_birth = models.DateField(blank=True, null=True)
    is_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    # Override username field
    username = models.CharField(
        max_length=150,
        unique=True,
        validators=[MinLengthValidator(3)]
    )
    
    class Meta:
        db_table = 'custom_users'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['email']),
            models.Index(fields=['username']),
        ]
    
    def __str__(self):
        return self.username
    
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}".strip()

# blog/models.py
class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name_plural = "categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    content = models.TextField()
    excerpt = models.TextField(max_length=500, blank=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')
    
    # Relationships
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True, blank=True)
    tags = models.ManyToManyField('Tag', blank=True, related_name='posts')
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)
    
    # SEO fields
    meta_title = models.CharField(max_length=200, blank=True)
    meta_description = models.TextField(max_length=300, blank=True)
    
    # Statistics
    view_count = models.PositiveIntegerField(default=0)
    like_count = models.PositiveIntegerField(default=0)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'published_at']),
            models.Index(fields=['author', 'status']),
            models.Index(fields=['category']),
        ]
        constraints = [
            models.CheckConstraint(
                check=models.Q(view_count__gte=0),
                name='positive_view_count'
            )
        ]
    
    def __str__(self):
        return self.title
    
    @property
    def is_published(self):
        return self.status == 'published'

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    color = models.CharField(max_length=7, default='#007bff')  # Hex color
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['name']
    
    def __str__(self):
        return self.name

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    parent = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='replies')
    
    content = models.TextField()
    is_approved = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['created_at']
        indexes = [
            models.Index(fields=['post', 'is_approved']),
        ]
    
    def __str__(self):
        return f"Comment by {self.author.username} on {self.post.title}"
```

## Spring Boot JPA Entities

### Equivalent JPA Entities
```java
// user/model/User.java
@Entity
@Table(name = "custom_users", 
       indexes = {
           @Index(name = "idx_user_email", columnList = "email"),
           @Index(name = "idx_user_username", columnList = "username")
       })
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 150)
    @Size(min = 3, max = 150, message = "Username must be between 3 and 150 characters")
    private String username;
    
    @Column(unique = true, nullable = false)
    @Email(message = "Invalid email format")
    private String email;
    
    @Column(name = "first_name", length = 30)
    private String firstName;
    
    @Column(name = "last_name", length = 150)
    private String lastName;
    
    @Column(name = "phone_number", length = 20)
    private String phoneNumber;
    
    @Column(name = "date_of_birth")
    private LocalDate dateOfBirth;
    
    @Column(name = "is_verified", nullable = false)
    @Builder.Default
    private Boolean isVerified = false;
    
    @Column(name = "is_active", nullable = false)
    @Builder.Default
    private Boolean isActive = true;
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Relationships
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Post> posts = new ArrayList<>();
    
    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Comment> comments = new ArrayList<>();
    
    // Business methods
    public String getFullName() {
        if (firstName == null && lastName == null) {
            return username;
        }
        return String.format("%s %s", 
                firstName != null ? firstName : "", 
                lastName != null ? lastName : "").trim();
    }
    
    @Override
    public String toString() {
        return username;
    }
}

// blog/model/Category.java
@Entity
@Table(name = "categories")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
public class Category {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false, length = 100)
    private String slug;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(name = "is_active", nullable = false)
    @Builder.Default
    private Boolean isActive = true;
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    // Relationships
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Post> posts = new ArrayList<>();
    
    @Override
    public String toString() {
        return name;
    }
}

// blog/model/Post.java
@Entity
@Table(name = "posts",
       indexes = {
           @Index(name = "idx_post_status_published", columnList = "status, published_at"),
           @Index(name = "idx_post_author_status", columnList = "author_id, status"),
           @Index(name = "idx_post_category", columnList = "category_id")
       })
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
@Check(constraints = "view_count >= 0")
public class Post {
    
    public enum Status {
        DRAFT("draft"),
        PUBLISHED("published"),
        ARCHIVED("archived");
        
        private final String value;
        
        Status(String value) {
            this.value = value;
        }
        
        public String getValue() {
            return value;
        }
    }
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String title;
    
    @Column(unique = true, nullable = false, length = 200)
    private String slug;
    
    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;
    
    @Column(length = 500)
    private String excerpt;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    @Builder.Default
    private Status status = Status.DRAFT;
    
    // Relationships
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;
    
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "post_tags",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    @Builder.Default
    private Set<Tag> tags = new HashSet<>();
    
    // Timestamps
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @Column(name = "published_at")
    private LocalDateTime publishedAt;
    
    // SEO fields
    @Column(name = "meta_title", length = 200)
    private String metaTitle;
    
    @Column(name = "meta_description", length = 300)
    private String metaDescription;
    
    // Statistics
    @Column(name = "view_count", nullable = false)
    @Builder.Default
    private Integer viewCount = 0;
    
    @Column(name = "like_count", nullable = false)
    @Builder.Default
    private Integer likeCount = 0;
    
    // Relationships
    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Comment> comments = new ArrayList<>();
    
    // Business methods
    public boolean isPublished() {
        return status == Status.PUBLISHED;
    }
    
    public void incrementViewCount() {
        this.viewCount++;
    }
    
    public void incrementLikeCount() {
        this.likeCount++;
    }
    
    @Override
    public String toString() {
        return title;
    }
}

// blog/model/Tag.java
@Entity
@Table(name = "tags")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
public class Tag {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false, length = 50)
    private String name;
    
    @Column(unique = true, nullable = false, length = 50)
    private String slug;
    
    @Column(length = 7)
    @Builder.Default
    private String color = "#007bff";
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    // Relationships
    @ManyToMany(mappedBy = "tags", fetch = FetchType.LAZY)
    @Builder.Default
    private Set<Post> posts = new HashSet<>();
    
    @Override
    public String toString() {
        return name;
    }
}

// blog/model/Comment.java
@Entity
@Table(name = "comments",
       indexes = {
           @Index(name = "idx_comment_post_approved", columnList = "post_id, is_approved")
       })
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EntityListeners(AuditingEntityListener.class)
public class Comment {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id", nullable = false)
    private User author;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Comment parent;
    
    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;
    
    @Column(name = "is_approved", nullable = false)
    @Builder.Default
    private Boolean isApproved = false;
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // Self-referencing relationship for replies
    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @Builder.Default
    private List<Comment> replies = new ArrayList<>();
    
    @Override
    public String toString() {
        return String.format("Comment by %s on %s", 
                author != null ? author.getUsername() : "Unknown",
                post != null ? post.getTitle() : "Unknown Post");
    }
}
```

## Repository Layer (Spring Data JPA)

### Django ORM vs Spring Data Repositories

#### Django QuerySet Operations
```python
# Django ORM queries
from django.db.models import Q, F, Count, Avg
from .models import User, Post, Comment

# Basic queries
users = User.objects.all()
active_users = User.objects.filter(is_active=True)
user = User.objects.get(email='user@example.com')

# Complex queries
published_posts = Post.objects.filter(
    status='published',
    published_at__lte=timezone.now()
).select_related('author', 'category').prefetch_related('tags')

# Aggregation
post_stats = Post.objects.aggregate(
    total_posts=Count('id'),
    avg_views=Avg('view_count'),
    total_views=Sum('view_count')
)

# Complex filtering
popular_posts = Post.objects.filter(
    Q(view_count__gt=1000) | Q(like_count__gt=100),
    status='published'
).annotate(
    comment_count=Count('comments')
).order_by('-view_count')

# Custom queries
recent_posts_by_category = Post.objects.raw('''
    SELECT p.*, COUNT(c.id) as comment_count
    FROM posts p
    LEFT JOIN comments c ON p.id = c.post_id
    WHERE p.created_at >= %s
    GROUP BY p.id
    ORDER BY p.created_at DESC
''', [one_week_ago])
```

#### Spring Data JPA Repositories
```java
// user/repository/UserRepository.java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Query methods (Spring Data generates implementation)
    Optional<User> findByEmail(String email);
    Optional<User> findByUsername(String username);
    List<User> findByIsActiveTrue();
    List<User> findByIsActiveTrueOrderByCreatedAtDesc();
    
    // Custom query with @Query annotation
    @Query("SELECT u FROM User u WHERE u.username LIKE %:query% OR u.email LIKE %:query%")
    List<User> searchUsers(@Param("query") String query);
    
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.posts WHERE u.id = :id")
    Optional<User> findByIdWithPosts(@Param("id") Long id);
    
    // Native SQL query
    @Query(value = "SELECT * FROM custom_users WHERE created_at >= :date", nativeQuery = true)
    List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);
    
    // Custom method with Specification
    @Query("SELECT u FROM User u WHERE " +
           "(:isActive is null OR u.isActive = :isActive) AND " +
           "(:query is null OR u.username LIKE %:query% OR u.email LIKE %:query%)")
    Page<User> findUsersWithFilters(@Param("isActive") Boolean isActive,
                                   @Param("query") String query,
                                   Pageable pageable);
    
    // Modifying queries
    @Modifying
    @Query("UPDATE User u SET u.isVerified = true WHERE u.id = :id")
    int verifyUser(@Param("id") Long id);
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.isActive = false AND u.createdAt < :date")
    int deleteInactiveUsersBefore(@Param("date") LocalDateTime date);
}

// blog/repository/PostRepository.java
@Repository
public interface PostRepository extends JpaRepository<Post, Long>, JpaSpecificationExecutor<Post> {
    
    // Basic queries
    List<Post> findByStatus(Post.Status status);
    List<Post> findByAuthor(User author);
    List<Post> findByCategory(Category category);
    
    // Complex queries with relationships
    @Query("SELECT p FROM Post p " +
           "JOIN FETCH p.author " +
           "LEFT JOIN FETCH p.category " +
           "LEFT JOIN FETCH p.tags " +
           "WHERE p.status = :status " +
           "ORDER BY p.createdAt DESC")
    List<Post> findPublishedPostsWithDetails(@Param("status") Post.Status status);
    
    // Aggregation queries
    @Query("SELECT p FROM Post p WHERE p.viewCount > :minViews OR p.likeCount > :minLikes")
    List<Post> findPopularPosts(@Param("minViews") Integer minViews, 
                               @Param("minLikes") Integer minLikes);
    
    @Query("SELECT COUNT(p) FROM Post p WHERE p.author = :author AND p.status = :status")
    long countPostsByAuthorAndStatus(@Param("author") User author, @Param("status") Post.Status status);
    
    // Pagination and sorting
    Page<Post> findByStatusOrderByCreatedAtDesc(Post.Status status, Pageable pageable);
    
    @Query("SELECT p FROM Post p WHERE " +
           "(:status is null OR p.status = :status) AND " +
           "(:categoryId is null OR p.category.id = :categoryId) AND " +
           "(:authorId is null OR p.author.id = :authorId) AND " +
           "(:query is null OR LOWER(p.title) LIKE LOWER(CONCAT('%', :query, '%')) OR " +
           "LOWER(p.content) LIKE LOWER(CONCAT('%', :query, '%')))")
    Page<Post> findPostsWithFilters(@Param("status") Post.Status status,
                                   @Param("categoryId") Long categoryId,
                                   @Param("authorId") Long authorId,
                                   @Param("query") String query,
                                   Pageable pageable);
    
    // Custom native query for complex operations
    @Query(value = """
        SELECT p.*, COUNT(c.id) as comment_count
        FROM posts p
        LEFT JOIN comments c ON p.id = c.post_id AND c.is_approved = true
        WHERE p.created_at >= :date
        GROUP BY p.id
        ORDER BY p.created_at DESC
        """, nativeQuery = true)
    List<Object[]> findRecentPostsWithCommentCount(@Param("date") LocalDateTime date);
    
    // Update operations
    @Modifying
    @Query("UPDATE Post p SET p.viewCount = p.viewCount + 1 WHERE p.id = :id")
    int incrementViewCount(@Param("id") Long id);
    
    @Modifying
    @Query("UPDATE Post p SET p.status = :newStatus WHERE p.author.id = :authorId AND p.status = :oldStatus")
    int updatePostStatusByAuthor(@Param("authorId") Long authorId, 
                                @Param("oldStatus") Post.Status oldStatus,
                                @Param("newStatus") Post.Status newStatus);
}

// blog/repository/CommentRepository.java
@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {
    
    List<Comment> findByPostAndIsApprovedTrueOrderByCreatedAt(Post post);
    List<Comment> findByAuthor(User author);
    List<Comment> findByParentIsNullAndPost(Post post);
    
    @Query("SELECT c FROM Comment c WHERE c.post.id = :postId AND c.isApproved = true AND c.parent IS NULL")
    List<Comment> findTopLevelCommentsByPostId(@Param("postId") Long postId);
    
    @Query("SELECT COUNT(c) FROM Comment c WHERE c.post = :post AND c.isApproved = true")
    long countApprovedCommentsByPost(@Param("post") Post post);
    
    @Modifying
    @Query("UPDATE Comment c SET c.isApproved = true WHERE c.id IN :ids")
    int approveComments(@Param("ids") List<Long> ids);
}
```

## Advanced Query Techniques

### Django Complex Queries
```python
# Complex Django queries
from django.db.models import Q, F, Count, Case, When, IntegerField

# Conditional annotations
posts_with_metrics = Post.objects.annotate(
    comment_count=Count('comments'),
    popularity_score=Case(
        When(view_count__gt=1000, then=F('view_count') * 2),
        When(view_count__gt=500, then=F('view_count') * 1.5),
        default=F('view_count'),
        output_field=IntegerField()
    )
).filter(status='published')

# Subqueries
from django.db.models import OuterRef, Subquery

latest_comment_date = Comment.objects.filter(
    post=OuterRef('pk')
).order_by('-created_at').values('created_at')[:1]

posts_with_latest_comment = Post.objects.annotate(
    latest_comment_date=Subquery(latest_comment_date)
).filter(status='published')

# Custom managers
class PublishedPostManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(
            status='published',
            published_at__lte=timezone.now()
        )

class Post(models.Model):
    # ... fields ...
    
    objects = models.Manager()  # Default manager
    published = PublishedPostManager()  # Custom manager
```

### Spring Data JPA Advanced Queries
```java
// Custom repository implementation
@Repository
public class PostRepositoryImpl {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    public List<PostWithMetrics> findPostsWithMetrics(PostSearchCriteria criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<PostWithMetrics> query = cb.createQuery(PostWithMetrics.class);
        Root<Post> post = query.from(Post.class);
        
        // Join with comments for count
        Join<Post, Comment> comments = post.join("comments", JoinType.LEFT);
        
        // Selection with computed fields
        query.select(cb.construct(
            PostWithMetrics.class,
            post.get("id"),
            post.get("title"),
            post.get("viewCount"),
            cb.count(comments),
            cb.selectCase()
                .when(cb.gt(post.get("viewCount"), 1000), cb.prod(post.get("viewCount"), 2))
                .when(cb.gt(post.get("viewCount"), 500), cb.prod(post.get("viewCount"), 1.5))
                .otherwise(post.get("viewCount"))
        ));
        
        // Conditions
        Predicate predicate = cb.conjunction();
        
        if (criteria.getStatus() != null) {
            predicate = cb.and(predicate, cb.equal(post.get("status"), criteria.getStatus()));
        }
        
        if (criteria.getAuthorId() != null) {
            predicate = cb.and(predicate, cb.equal(post.get("author").get("id"), criteria.getAuthorId()));
        }
        
        if (criteria.getMinViews() != null) {
            predicate = cb.and(predicate, cb.ge(post.get("viewCount"), criteria.getMinViews()));
        }
        
        query.where(predicate);
        query.groupBy(post.get("id"));
        query.orderBy(cb.desc(post.get("createdAt")));
        
        return entityManager.createQuery(query).getResultList();
    }
    
    // Subquery example
    public List<Post> findPostsWithRecentComments(LocalDateTime since) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<Post> query = cb.createQuery(Post.class);
        Root<Post> post = query.from(Post.class);
        
        // Subquery for latest comment date
        Subquery<LocalDateTime> subquery = query.subquery(LocalDateTime.class);
        Root<Comment> comment = subquery.from(Comment.class);
        subquery.select(cb.greatest(comment.get("createdAt")))
               .where(cb.equal(comment.get("post"), post));
        
        query.select(post)
             .where(cb.greaterThan(subquery, since))
             .orderBy(cb.desc(post.get("createdAt")));
        
        return entityManager.createQuery(query).getResultList();
    }
}

// Specifications for dynamic queries
public class PostSpecifications {
    
    public static Specification<Post> hasStatus(Post.Status status) {
        return (root, query, criteriaBuilder) ->
            status == null ? null : criteriaBuilder.equal(root.get("status"), status);
    }
    
    public static Specification<Post> hasAuthor(Long authorId) {
        return (root, query, criteriaBuilder) ->
            authorId == null ? null : criteriaBuilder.equal(root.get("author").get("id"), authorId);
    }
    
    public static Specification<Post> hasCategory(Long categoryId) {
        return (root, query, criteriaBuilder) ->
            categoryId == null ? null : criteriaBuilder.equal(root.get("category").get("id"), categoryId);
    }
    
    public static Specification<Post> titleOrContentContains(String searchText) {
        return (root, query, criteriaBuilder) -> {
            if (searchText == null || searchText.trim().isEmpty()) {
                return null;
            }
            String pattern = "%" + searchText.toLowerCase() + "%";
            return criteriaBuilder.or(
                criteriaBuilder.like(criteriaBuilder.lower(root.get("title")), pattern),
                criteriaBuilder.like(criteriaBuilder.lower(root.get("content")), pattern)
            );
        };
    }
    
    public static Specification<Post> createdAfter(LocalDateTime date) {
        return (root, query, criteriaBuilder) ->
            date == null ? null : criteriaBuilder.greaterThan(root.get("createdAt"), date);
    }
    
    public static Specification<Post> withMinimumViews(Integer minViews) {
        return (root, query, criteriaBuilder) ->
            minViews == null ? null : criteriaBuilder.ge(root.get("viewCount"), minViews);
    }
}

// Usage in service
@Service
public class PostService {
    
    private final PostRepository postRepository;
    
    public Page<Post> searchPosts(PostSearchCriteria criteria, Pageable pageable) {
        Specification<Post> spec = Specification.where(PostSpecifications.hasStatus(criteria.getStatus()))
                .and(PostSpecifications.hasAuthor(criteria.getAuthorId()))
                .and(PostSpecifications.hasCategory(criteria.getCategoryId()))
                .and(PostSpecifications.titleOrContentContains(criteria.getSearchText()))
                .and(PostSpecifications.createdAfter(criteria.getCreatedAfter()))
                .and(PostSpecifications.withMinimumViews(criteria.getMinViews()));
        
        return postRepository.findAll(spec, pageable);
    }
}
```

## Entity Lifecycle and Auditing

### Django Model Lifecycle
```python
# Django signals for model lifecycle
from django.db.models.signals import pre_save, post_save, pre_delete, post_delete
from django.dispatch import receiver
from django.utils.text import slugify

@receiver(pre_save, sender=Post)
def generate_slug(sender, instance, **kwargs):
    if not instance.slug:
        instance.slug = slugify(instance.title)

@receiver(post_save, sender=Post)
def update_author_post_count(sender, instance, created, **kwargs):
    if created:
        # Update author's post count cache
        cache.delete(f"user_post_count_{instance.author.id}")

@receiver(pre_delete, sender=Post)
def cleanup_related_data(sender, instance, **kwargs):
    # Clean up related files, cache, etc.
    instance.tags.clear()
```

### Spring Boot Entity Lifecycle
```java
// JPA Entity Listeners
@EntityListeners(AuditingEntityListener.class)
@Entity
public class Post {
    // ... fields ...
    
    @PrePersist
    protected void onCreate() {
        if (slug == null || slug.trim().isEmpty()) {
            slug = generateSlug(title);
        }
    }
    
    @PreUpdate
    protected void onUpdate() {
        // Any pre-update logic
    }
    
    @PostPersist
    protected void afterCreate() {
        // Post-creation logic
        log.info("Post created: {}", title);
    }
    
    @PostUpdate
    protected void afterUpdate() {
        // Post-update logic
    }
    
    @PreRemove
    protected void beforeDelete() {
        // Pre-deletion cleanup
        tags.clear();
    }
    
    private String generateSlug(String title) {
        return title.toLowerCase()
                   .replaceAll("[^a-z0-9\\s]", "")
                   .replaceAll("\\s+", "-")
                   .trim();
    }
}

// Global entity listener
@Component
public class PostEntityListener {
    
    @Autowired
    private CacheManager cacheManager;
    
    @PostPersist
    @PostUpdate
    @PostRemove
    public void invalidateCache(Post post) {
        Cache cache = cacheManager.getCache("posts");
        if (cache != null) {
            cache.evict("user_posts_" + post.getAuthor().getId());
            cache.evict("category_posts_" + (post.getCategory() != null ? post.getCategory().getId() : "null"));
        }
    }
}

// Auditing configuration
@Configuration
@EnableJpaAuditing
public class AuditingConfig {
    
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> {
            // Get current user from security context
            Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
            if (authentication == null || !authentication.isAuthenticated()) {
                return Optional.of("system");
            }
            return Optional.of(authentication.getName());
        };
    }
}

// Auditable base entity
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
@Data
public abstract class AuditableEntity {
    
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}
```

## Performance Optimization

### Django Query Optimization
```python
# Django optimization techniques
from django.db.models import Prefetch

# Select related for foreign keys
posts = Post.objects.select_related('author', 'category').all()

# Prefetch related for many-to-many and reverse foreign keys
posts = Post.objects.prefetch_related('tags', 'comments__author').all()

# Custom prefetch
approved_comments = Comment.objects.filter(is_approved=True)
posts = Post.objects.prefetch_related(
    Prefetch('comments', queryset=approved_comments, to_attr='approved_comments')
).all()

# Only/defer for field selection
posts = Post.objects.only('id', 'title', 'created_at').all()
posts = Post.objects.defer('content').all()

# Database functions
from django.db.models import F, Value
from django.db.models.functions import Concat

posts = Post.objects.annotate(
    full_title=Concat('title', Value(' - '), 'category__name')
).all()
```

### Spring Boot JPA Optimization
```java
// JPA optimization techniques

// Entity graphs for fetch optimization
@NamedEntityGraph(
    name = "Post.withAuthorAndCategory",
    attributeNodes = {
        @NamedAttributeNode("author"),
        @NamedAttributeNode("category"),
        @NamedAttributeNode(value = "tags")
    }
)
@Entity
public class Post {
    // ... entity definition
}

// Repository with entity graph
@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
    
    @EntityGraph("Post.withAuthorAndCategory")
    @Query("SELECT p FROM Post p WHERE p.status = :status")
    List<Post> findPostsWithDetails(@Param("status") Post.Status status);
    
    // Projection for limited fields
    @Query("SELECT p.id, p.title, p.createdAt FROM Post p WHERE p.status = :status")
    List<PostSummary> findPostSummaries(@Param("status") Post.Status status);
}

// Projection interface
public interface PostSummary {
    Long getId();
    String getTitle();
    LocalDateTime getCreatedAt();
}

// Batch operations
@Repository
public class PostRepositoryImpl {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Transactional
    public void batchUpdateViewCounts(List<Long> postIds, int increment) {
        String jpql = "UPDATE Post p SET p.viewCount = p.viewCount + :increment WHERE p.id IN :ids";
        entityManager.createQuery(jpql)
                     .setParameter("increment", increment)
                     .setParameter("ids", postIds)
                     .executeUpdate();
    }
    
    @Transactional
    public void batchInsertPosts(List<Post> posts) {
        int batchSize = 20;
        for (int i = 0; i < posts.size(); i++) {
            entityManager.persist(posts.get(i));
            if (i % batchSize == 0 && i > 0) {
                entityManager.flush();
                entityManager.clear();
            }
        }
    }
}

// Caching
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Category {
    // ... entity definition
}

// Query result caching
@Repository
public interface CategoryRepository extends JpaRepository<Category, Long> {
    
    @QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
    @Query("SELECT c FROM Category c WHERE c.isActive = true ORDER BY c.name")
    List<Category> findActiveCategories();
}
```

## Migration Summary

| Aspect | Django | Spring Boot JPA |
|--------|--------|-----------------|
| **Model Definition** | Python classes | Java classes with annotations |
| **Fields** | Model fields | Entity attributes with annotations |
| **Relationships** | ForeignKey, ManyToMany | @OneToMany, @ManyToMany, etc. |
| **Validation** | Model validators | Bean Validation annotations |
| **Queries** | QuerySet methods | Repository methods + JPQL |
| **Raw SQL** | .raw() method | @Query with nativeQuery = true |
| **Aggregation** | .aggregate() | @Query with aggregate functions |
| **Caching** | Django cache framework | Hibernate 2nd level cache |
| **Lifecycle** | Django signals | JPA lifecycle callbacks |
| **Migration** | Django migrations | Flyway/Liquibase |

The key transition is moving from Django's QuerySet-based ORM to JPA's repository pattern with explicit relationship mapping and type-safe queries.