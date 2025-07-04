# HTTP Status Codes

Each HTTP request response carries a numeric status code that indicates the result of the requested operation. REST makes intensive use of standard HTTP status codes to communicate the result uniformly.

## Status Code Categories

HTTP codes are grouped by hundreds:

- **1xx (Informational)**: Request received, continuing process
- **2xx (Success)**: Request successfully received, understood, and accepted
- **3xx (Redirection)**: Further action needed to complete request
- **4xx (Client Error)**: Request contains bad syntax or cannot be fulfilled
- **5xx (Server Error)**: Server failed to fulfill a valid request

## 2xx Success Codes

### 200 OK
**The request was successful** and (if it was GET) the resource is returned in the body.

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

**When to use**: Successful GET, PUT, PATCH requests

### 201 Created
**Resource created successfully** (typically used after a POST). It usually also includes the URL of the new resource in the Location header.

```http
HTTP/1.1 201 Created
Location: /users/124
Content-Type: application/json

{
  "id": 124,
  "name": "Bob",
  "email": "bob@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**When to use**: Successful POST requests that create new resources

### 204 No Content
**Successful request but without response body**. Used when the server successfully processed the request but there's no content to return.

```http
HTTP/1.1 204 No Content
```

**When to use**: 
- Successful DELETE operations
- PUT/PATCH operations where you don't need to return the updated resource
- Operations that don't require a response body

## 4xx Client Error Codes

### 400 Bad Request
**The request is invalid or malformed**. The server cannot understand the request due to invalid syntax.

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Bad Request",
  "message": "Missing required field: email",
  "details": {
    "field": "email",
    "issue": "required"
  }
}
```

**Common causes**:
- Missing required fields
- Invalid data types
- Malformed JSON
- Invalid query parameters

### 401 Unauthorized
**Authentication is required** and has failed or has not been provided.

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer realm="example"
Content-Type: application/json

{
  "error": "Unauthorized",
  "message": "Authentication required"
}
```

**When to use**: User is not authenticated (no token, invalid token, expired token)

### 403 Forbidden
**The server understood the request but refuses to authorize it**. The client is authenticated but doesn't have permission.

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": "Forbidden",
  "message": "Insufficient permissions to access this resource"
}
```

**When to use**: User is authenticated but lacks permission for the specific resource/action

### 404 Not Found
**The requested resource doesn't exist** at the specified URL.

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": "Not Found",
  "message": "User with ID 999 not found"
}
```

**When to use**: 
- Resource doesn't exist
- Invalid ID in URL
- Endpoint doesn't exist

### 409 Conflict
**The request conflicts with the current state of the server**. Often used for duplicate resources.

```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": "Conflict",
  "message": "User with email 'alice@example.com' already exists"
}
```

**When to use**: 
- Duplicate email addresses
- Concurrent modifications
- Business rule violations

### 422 Unprocessable Entity
**The request was well-formed but contains semantic errors**. The server understands the content type and syntax but cannot process the contained instructions.

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
  "error": "Unprocessable Entity",
  "message": "Validation failed",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    },
    {
      "field": "age",
      "message": "Age must be between 18 and 100"
    }
  ]
}
```

**When to use**: Validation errors, business logic violations

## 5xx Server Error Codes

### 500 Internal Server Error
**Server error when processing the request**. Something went wrong on the server side.

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

**When to use**: 
- Database connection failures
- Unhandled exceptions
- Server configuration issues

### 502 Bad Gateway
**The server received an invalid response from an upstream server**.

```http
HTTP/1.1 502 Bad Gateway
Content-Type: application/json

{
  "error": "Bad Gateway",
  "message": "Unable to connect to database service"
}
```

**When to use**: 
- API gateway issues
- Microservice communication failures
- Third-party service unavailability

### 503 Service Unavailable
**The server is temporarily unable to handle the request**.

```http
HTTP/1.1 503 Service Unavailable
Retry-After: 60
Content-Type: application/json

{
  "error": "Service Unavailable",
  "message": "Server is under maintenance"
}
```

**When to use**: 
- Server maintenance
- Overloaded servers
- Temporary outages

## Best Practices

### 1. Be Consistent
Use the same status codes for similar situations across your API:

```http
# Always use 201 for successful creation
POST /users → 201 Created

# Always use 204 for successful deletion
DELETE /users/123 → 204 No Content

# Always use 404 for missing resources
GET /users/999 → 404 Not Found
```

### 2. Provide Meaningful Error Messages
Include helpful information in error responses:

```json
{
  "error": "Validation Error",
  "message": "Please check your input",
  "details": [
    {
      "field": "email",
      "message": "Email must be a valid format",
      "value": "invalid-email"
    }
  ],
  "timestamp": "2024-01-15T10:30:00Z",
  "request_id": "req_123456"
}
```

### 3. Use Appropriate Status Codes
Don't overuse generic codes like 200 or 500:

```http
# ✅ Good
POST /users → 201 Created
DELETE /users/123 → 204 No Content

# ❌ Avoid
POST /users → 200 OK (with created user)
DELETE /users/123 → 200 OK (with success message)
```

### 4. Include Headers When Appropriate
Add relevant headers to responses:

```http
HTTP/1.1 201 Created
Location: /users/124
Content-Type: application/json
X-Rate-Limit-Remaining: 999

{
  "id": 124,
  "name": "Bob"
}
```

## Common Status Code Patterns

| Operation | Success | Error |
|-----------|---------|-------|
| GET (single) | 200 OK | 404 Not Found |
| GET (collection) | 200 OK | 400 Bad Request |
| POST (create) | 201 Created | 400/409/422 |
| PUT (update) | 200 OK | 404 Not Found |
| PATCH (partial) | 200 OK | 404 Not Found |
| DELETE | 204 No Content | 404 Not Found |

## Summary

- **2xx**: Success - request completed successfully
- **4xx**: Client error - something wrong with the request
- **5xx**: Server error - something wrong on the server
- **Be consistent** with status code usage
- **Provide helpful error messages** with details
- **Use appropriate codes** for each situation

## Next Steps

In the next section, we'll explore the tools that make working with REST APIs easier: Postman and Swagger. 