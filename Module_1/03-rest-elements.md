# Elements of a RESTful API

A REST API leverages the HTTP protocol as the communication base. Let's look at the key elements of how a typical interaction works:

## Resource and URL

In REST, all exposed information is modeled as **resources**, which are identifiable entities. Each resource is uniquely identified by a URI (Uniform Resource Identifier), usually a descriptive URL.

### Resource Naming Conventions

By convention, use **plural nouns** for resource names to maintain uniformity and resource-oriented semantics:

```
✅ Good Examples:
https://api.example.com/users
https://api.example.com/orders
https://api.example.com/products

❌ Avoid:
https://api.example.com/getUsers
https://api.example.com/user_list
https://api.example.com/fetch_orders
```

### URL Structure Examples

```http
# Collection of resources
GET /users                    # Get all users
GET /users?page=2&limit=10    # Get users with pagination

# Specific resource
GET /users/123                # Get user with ID 123
PUT /users/123                # Update user 123
DELETE /users/123             # Delete user 123

# Nested resources
GET /users/123/orders         # Get orders for user 123
GET /users/123/orders/456     # Get specific order 456 for user 123
```

### Hierarchical Relationships

URLs can represent relationships between resources:

```
/users/123/orders/456/items/789
```

This represents: Item 789 in Order 456 belonging to User 123.

## HTTP Verbs (Methods)

REST relies on a limited set of standard HTTP methods to operate on resources. The four fundamental methods (mapped to classic CRUD operations) are:

### 1. GET - Read Operations

**Purpose**: Retrieve a resource or list of resources

```http
GET /users              # Get all users
GET /users/123          # Get specific user
GET /users?name=alice   # Get users with filter
```

**Characteristics**:
- Safe and idempotent
- Usually no request body
- Can be cached
- Returns data in response body

### 2. POST - Create Operations

**Purpose**: Create a new resource

```http
POST /users
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}
```

**Characteristics**:
- Not idempotent (multiple calls create multiple resources)
- Usually has request body
- Returns 201 Created on success
- Often returns the created resource

### 3. PUT - Update Operations

**Purpose**: Completely update an existing resource

```http
PUT /users/123
Content-Type: application/json

{
  "id": 123,
  "name": "Alice Updated",
  "email": "alice.updated@example.com",
  "age": 30
}
```

**Characteristics**:
- Idempotent (multiple calls have same effect)
- Replaces entire resource
- Returns 200 OK or 204 No Content
- Must include all fields

### 4. DELETE - Delete Operations

**Purpose**: Delete an existing resource

```http
DELETE /users/123
```

**Characteristics**:
- Idempotent
- Usually no request body
- Returns 204 No Content on success
- Resource becomes unavailable

### Additional HTTP Methods

#### PATCH - Partial Updates

**Purpose**: Update specific fields of a resource

```http
PATCH /users/123
Content-Type: application/json

{
  "email": "newemail@example.com"
}
```

#### HEAD - Metadata Only

**Purpose**: Get headers without body (useful for checking if resource exists)

```http
HEAD /users/123
```

#### OPTIONS - Available Operations

**Purpose**: Get information about available operations

```http
OPTIONS /users
```

## Body and Data Format

### Request Body

- **GET requests**: Usually no body (data in URL parameters)
- **POST/PUT/PATCH requests**: Include body with resource data
- **DELETE requests**: Usually no body

### Response Body

Server responses typically include a body with the requested resource or operation result.

### JSON Format (Most Common)

```json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z",
  "active": true
}
```

### Content Negotiation

Both client and server agree on format using HTTP headers:

```http
# Client requests JSON
GET /users/123
Accept: application/json

# Server responds with JSON
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Alice"
}
```

### Alternative Formats

While JSON is most common, REST APIs can support multiple formats:

```http
# XML
Accept: application/xml
Content-Type: application/xml

# Plain text
Accept: text/plain
Content-Type: text/plain

# HTML
Accept: text/html
Content-Type: text/html
```

## HTTP Headers

Headers provide metadata for requests and responses:

### Common Request Headers

```http
GET /users HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
User-Agent: MyApp/1.0
```

### Common Response Headers

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600
ETag: "33a64df551"
X-Rate-Limit-Remaining: 999
```

### Important Headers Explained

| Header | Purpose | Example |
|--------|---------|---------|
| `Accept` | What format client wants | `Accept: application/json` |
| `Content-Type` | Format of request/response body | `Content-Type: application/json` |
| `Authorization` | Authentication credentials | `Authorization: Bearer token` |
| `Cache-Control` | Caching instructions | `Cache-Control: max-age=3600` |
| `ETag` | Resource version identifier | `ETag: "33a64df551"` |

## Summary of Operation Flow

1. **Client** makes HTTP request to API URL
2. **Server** receives and processes request
3. **Server** interacts with database/other systems
4. **Server** returns HTTP response with status code and body
5. **Client** processes response

### Example Complete Flow

```http
# Request
GET /users/123 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Response
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Key Takeaways

- **Resources** are identified by URLs
- **HTTP methods** define operations (GET, POST, PUT, DELETE)
- **JSON** is the most common data format
- **Headers** provide metadata and context
- **Stateless** - each request is independent
- **Uniform interface** - consistent patterns across all resources

## Next Steps

In the next section, we'll explore HTTP status codes and how they communicate the result of API operations. 