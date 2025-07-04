# REST API Principles

An API is considered RESTful if it complies with the principles or architectural constraints defined in REST. These principles ensure that the API has the desirable properties of simplicity, scalability, and flexibility. The main ones are:

## 1. Client-Server Architecture

**Separation of concerns** between the client (consumer) and the server (resource provider). The client requests resources and the server provides them; both evolve independently as long as the interface (API contract) remains uniform.

### Why This Matters
- Allows independent evolution of client and server
- Enables different teams to work on frontend and backend
- Supports multiple client types (web, mobile, desktop) using the same API

## 2. Stateless Communication

Each HTTP request from the client to the server must contain **all necessary information**, without requiring the server to store client session data. This means the server doesn't maintain state between requests (stateless).

### Key Characteristics
- No server-side session storage
- Each request is independent
- All context must be included in the request
- Improves scalability and reliability

### Example
```http
GET /users/123 HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

The server doesn't need to remember who made the request - the token contains all necessary authentication information.

## 3. Cacheable

API responses must indicate whether they are **cacheable or not**, so that when possible, the client (or intermediaries) can store responses and reuse them in future requests.

### Benefits
- Reduces server load
- Improves response times
- Reduces bandwidth usage
- Better user experience

### Implementation
```http
HTTP/1.1 200 OK
Cache-Control: max-age=3600
ETag: "33a64df551"
```

## 4. Uniform Interface

This is perhaps the **cornerstone of REST**. It implies that all API resources are handled consistently and standardly.

### Four Interface Constraints

#### 4.1 Resource Identification
Resources are identified by URIs (URLs):
```
https://api.example.com/users
https://api.example.com/users/123
https://api.example.com/users/123/orders
```

#### 4.2 Manipulation Through Representations
Resources are manipulated through their representations (JSON, XML, etc.):
```json
{
  "id": 123,
  "name": "Alice",
  "email": "alice@example.com"
}
```

#### 4.3 Self-Descriptive Messages
Each message contains enough information to be processed:
```http
GET /users HTTP/1.1
Accept: application/json
Content-Type: application/json
```

#### 4.4 HATEOAS (Hypermedia as the Engine of Application State)
Resources contain links to related resources:
```json
{
  "id": 123,
  "name": "Alice",
  "links": [
    {"rel": "self", "href": "/users/123"},
    {"rel": "orders", "href": "/users/123/orders"}
  ]
}
```

## 5. Layered System

The architecture can be composed of **hierarchical layers** invisible to the client (e.g., load balancers, proxies, gateways, etc. between the client and the final server).

### Benefits
- **Security**: Add authentication/authorization layers
- **Scalability**: Add load balancers
- **Caching**: Add CDN or proxy caches
- **Monitoring**: Add logging and analytics layers

### Example Architecture
```
Client → Load Balancer → API Gateway → Authentication → API Server → Database
```

## 6. Code on Demand (Optional)

This is the **only optional principle**. It means that servers can extend client functionality by sending executable code in responses, for example JavaScript scripts that the client can execute.

### When It's Used
- Web applications with dynamic JavaScript
- Progressive Web Apps (PWAs)
- Micro-frontend architectures

### Example
```html
<script src="https://api.example.com/scripts/widget.js"></script>
```

## Summary

If an API fulfills all these principles (except the last one, which is optional), we can consider it a **complete RESTful API**. Despite this, many real-world implementations self-identify as "REST" by only partially fulfilling these principles.

### Key Takeaways
- **Stateless**: Each request is independent
- **Cacheable**: Responses can be stored and reused
- **Uniform Interface**: Consistent resource handling
- **Layered**: Multiple layers can be added transparently
- **Client-Server**: Clear separation of concerns

## Next Steps

In the next section, we'll explore how these principles are applied in practice through the elements of a RESTful API. 