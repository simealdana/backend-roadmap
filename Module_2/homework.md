# Advanced Assignment – Full-Stack TODO API with Authentication and Database

## Objective
Transform the basic TODO API from Module 1 into a production-ready application with user authentication, database persistence, middlewares, and advanced features. This assignment will test your understanding of Django, REST APIs, authentication, and database integration.

---

## Project Overview
Build a multi-user TODO application where each user can manage their own tasks. The system must include user registration, authentication, authorization, and advanced task management features.

---

## Core Features

### 1. User Management
- **User Registration**: `/api/auth/register/`
- **User Login**: `/api/auth/login/`
- **User Logout**: `/api/auth/logout/`
- **User Profile**: `/api/auth/profile/`

### 2. Task Management (Enhanced from Module 1)
Each task should have the following structure:

```json
{
  "id": 1,
  "title": "Buy milk",
  "description": "Go to the store and buy skimmed milk",
  "completed": false,
  "priority": "high", // possible values: low, medium, high
  "createdAt": "2025-07-04T12:30:00Z",
  "dueDate": "2025-07-05T23:59:00Z",
  "tags": ["shopping", "urgent"],
  "locked": false,
  "userId": 1,
  "category": "personal", // possible values: personal, work, health, finance
  "estimatedHours": 2.5,
  "actualHours": 0,
  "attachments": ["file1.pdf", "file2.jpg"],
  "sharedWith": [2, 3], // array of user IDs
  "parentTask": null, // for subtasks
  "subtasks": [4, 5] // array of subtask IDs
}
```

---

## Required Endpoints

### Authentication Endpoints
| Method | Route                    | Description                                    |
|--------|--------------------------|------------------------------------------------|
| POST   | `/api/auth/register/`    | Register a new user                            |
| POST   | `/api/auth/login/`       | Login user and return JWT token               |
| POST   | `/api/auth/logout/`      | Logout user (invalidate token)                |
| GET    | `/api/auth/profile/`     | Get current user profile                      |
| PUT    | `/api/auth/profile/`     | Update user profile                           |

### Task Endpoints (All require authentication)
| Method | Route                    | Description                                    |
|--------|--------------------------|------------------------------------------------|
| GET    | `/api/tasks/`            | List user's tasks with filters                |
| GET    | `/api/tasks/:id/`        | Get task details                              |
| POST   | `/api/tasks/`            | Create a new task                             |
| PUT    | `/api/tasks/:id/`        | Fully replace a task                          |
| PATCH  | `/api/tasks/:id/`        | Partially update a task                       |
| DELETE | `/api/tasks/:id/`        | Delete a task                                 |
| POST   | `/api/tasks/:id/share/`  | Share task with other users                   |
| POST   | `/api/tasks/:id/subtask/`| Create a subtask                              |

### Advanced Endpoints
| Method | Route                    | Description                                    |
|--------|--------------------------|------------------------------------------------|
| GET    | `/api/tasks/shared/`     | Get tasks shared with current user            |
| GET    | `/api/tasks/analytics/`  | Get task completion statistics                |
| POST   | `/api/tasks/bulk/`       | Create multiple tasks at once                 |
| PUT    | `/api/tasks/bulk/`       | Update multiple tasks at once                 |

---

## Technical Requirements

### 1. Database Integration
- Use **PostgreSQL** with Django ORM
- Implement proper database migrations
- Use Django models with relationships
- Include database indexes for performance

### 2. Authentication & Authorization
- Implement **JWT-based authentication**
- Use Django REST Framework with JWT tokens
- Implement proper token refresh mechanism
- Add role-based access control (Admin, User)

### 3. Middlewares Implementation
Create and implement the following custom middlewares:

#### Required Middlewares:
- **`RequestLoggingMiddleware`**: Log all API requests with user info, IP, timestamp, and response time
- **`RateLimitingMiddleware`**: Limit API requests per user (e.g., 100 requests per hour)
- **`CorsMiddleware`**: Handle Cross-Origin Resource Sharing
- **`AuthenticationMiddleware`**: Verify JWT tokens and set user context
- **`ErrorHandlingMiddleware`**: Centralized error handling and logging

#### Optional Middlewares:
- **`AnalyticsMiddleware`**: Track user behavior and task completion patterns
- **`CacheMiddleware`**: Cache frequently accessed data
- **`SecurityMiddleware`**: Add security headers and prevent common attacks

### 4. Advanced Features

#### Task Filtering & Search
Support multiple query parameters:
- `completed` (boolean)
- `priority` (low, medium, high)
- `category` (personal, work, health, finance)
- `tags` (comma-separated)
- `overdue` (boolean)
- `shared` (boolean)
- `search` (text search in title and description)
- `date_from` and `date_to` (date range)
- `estimated_hours_min` and `estimated_hours_max`

#### Task Sharing & Collaboration
- Users can share tasks with other users
- Shared tasks appear in both users' task lists
- Implement read/write permissions for shared tasks
- Notifications when tasks are shared

#### Subtasks & Task Hierarchy
- Tasks can have subtasks (parent-child relationship)
- Complete parent task only when all subtasks are done
- Calculate progress percentage for tasks with subtasks

#### Analytics & Reporting
- Task completion statistics by user
- Time tracking (estimated vs actual hours)
- Category-wise task distribution
- Overdue task analysis

---

## Database Schema

### User Model
```python
class User(AbstractUser):
    email = models.EmailField(unique=True)
    profile_picture = models.URLField(blank=True)
    timezone = models.CharField(max_length=50, default='UTC')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### Task Model
```python
class Task(models.Model):
    PRIORITY_CHOICES = [('low', 'Low'), ('medium', 'Medium'), ('high', 'High')]
    CATEGORY_CHOICES = [('personal', 'Personal'), ('work', 'Work'), ('health', 'Health'), ('finance', 'Finance')]
    
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='tasks')
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    priority = models.CharField(max_length=10, choices=PRIORITY_CHOICES, default='medium')
    category = models.CharField(max_length=20, choices=CATEGORY_CHOICES, default='personal')
    due_date = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    locked = models.BooleanField(default=False)
    estimated_hours = models.DecimalField(max_digits=5, decimal_places=2, null=True, blank=True)
    actual_hours = models.DecimalField(max_digits=5, decimal_places=2, default=0)
    parent_task = models.ForeignKey('self', on_delete=models.CASCADE, null=True, blank=True, related_name='subtasks')
    shared_with = models.ManyToManyField(User, related_name='shared_tasks', blank=True)
```

### Tag Model
```python
class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    color = models.CharField(max_length=7, default='#007bff')  # Hex color
    created_by = models.ForeignKey(User, on_delete=models.CASCADE)
```

### TaskTag Model (Many-to-Many relationship)
```python
class TaskTag(models.Model):
    task = models.ForeignKey(Task, on_delete=models.CASCADE)
    tag = models.ForeignKey(Tag, on_delete=models.CASCADE)
```



## Security Requirements

### 1. Authentication Security
- Secure JWT token storage
- Token refresh mechanism
- Password strength validation
- Account lockout after failed attempts

### 2. Authorization
- Users can only access their own tasks
- Proper permission checks for shared tasks
- Role-based access control

### 3. Data Protection
- Input validation and sanitization
- SQL injection prevention


### 4. API Security
- Rate limiting
- Request size limits
- Secure headers
- HTTPS enforcement

