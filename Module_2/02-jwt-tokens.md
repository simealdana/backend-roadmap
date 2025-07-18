


# JSON Web Tokens (JWT)

A **JSON Web Token (JWT)** is an open standard (RFC 7519) based on JSON for representing verified information compactly and securely between two parties. Essentially, it's a token that contains **"claims"** (statements) about a user or other subject, digitally signed. JWT was proposed by the IETF (RFC 7519) as a **self-contained** means of transmitting authentication credentials or data between systems. For example, after a successful login, a server can generate a JWT that indicates the user's identity and permissions; the client then includes that token in each request, so that the receiving server can verify it without needing to maintain sessions in memory. JWTs are designed to be **compact** (URL-safe) and **portable**, making it easy to send them in HTTP headers, parameters, or even in the URL.

A JWT consists of three main parts separated by dots (`.`): the header, the payload, and the signature. Each part is encoded in Base64URL. In its human-readable form, a JWT would look like this:

```
xxxxxxxxxx.yyyyyyyyyy.zzzzzzzzzz
```

where each long segment corresponds to one of the encoded parts. Below we show an example of an encoded JWT token (its three parts have been highlighted with different colors):

*Figure: Example of an encoded JSON Web Token (JWT) in Base64. The three parts (Header, Payload, and Signature) appear separated by dots.* This compact format facilitates its transmission in an HTTP header or URL.

## What is JWT used for and why was it created?

The most common use of JWT is for **authentication** of users in APIs and web applications. When the user logs in, the server creates a JWT that encapsulates their identity and permissions; the client stores that token and sends it in each request (for example, in the header `Authorization: Bearer <token>`). This way, the receiving server validates the token's signature and claims to authorize the request without needing to look up the user in a server session. This "stateless" mechanism eliminates the need to store sessions on the server, which simplifies system scalability.

In addition to authentication, JWTs facilitate **Single Sign-On (SSO)** and secure information exchange between different domains or services. For example, the same JWT can serve to authenticate the user across multiple subdomains or different microservices without re-sending it to the original login server. Another typical situation is *trusted data exchange*: since the JWT signature guarantees that it hasn't been modified, both parties can trust the payload's integrity. In summary, JWT was created to offer a standard, compact, and secure method of transmitting credentials and claims between clients and servers, especially useful in modern architectures of APIs, microservices, mobile applications, and Single Sign-On.

## Internal Structure of a JWT

A JWT token is divided into three main parts, each encoded in Base64URL and separated by dots:

| Part | Content (example) |
|------|-------------------|
| **Header** | A JSON object that indicates the token type and signature algorithm. Example: |

```json
{"alg":"HS256","typ":"JWT"}
```

Here `alg` is the algorithm (e.g., HS256 for HMAC-SHA256) and `typ` indicates it's a JWT. |
| **Payload** | Contains the **claims** about the entity (usually the user) and metadata. It can include common reserved fields, for example: |

```json
{"sub":"1234567890","name":"John Doe","admin":true,"iat":1516239022}
```

Here `sub` is the subject identifier, `name` could be the username, `admin` is a custom field, and `iat` (issued at) indicates when it was issued. Among the reserved claims, it's recommended to use `iss` (issuer), `exp` (expiration), `sub` (subject), `aud` (audience), etc. Other fields can be added as needed (roles, permissions, profile data, etc.), always avoiding including overly sensitive information. |
| **Signature** | To create the signature, the encoded header and payload are taken (separated by a dot), the indicated algorithm is applied with a secret key (or private key), and the result is encoded in Base64URL. For example, using HMAC-SHA256: |

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret)
```

The result is the third part of the JWT. The signature allows the receiver to verify that the token was issued by who it claims (with the correct key) and that it hasn't been altered in transit. |

Together, an encoded JWT might look like this (each segment corresponds to the three parts):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9   // Encoded header
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9   // Encoded payload
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c     // Generated signature
```

*(In this example, the real secret was omitted for simplicity).*

## JWT Creation and Verification

The **generation process** and use of a JWT typically follows these steps:

1. **Login:** The user sends their credentials to the authentication server.
2. **Generation:** The server verifies the credentials. If valid, it creates the JWT by first building the header and payload with the desired information (claims), then calculates the signature with the secret key.
3. **Sending to client:** The server returns the JWT token to the client (e.g., in the HTTP response).
4. **Storage:** The client stores the JWT securely (e.g., in an `HttpOnly` cookie or local storage) and includes it in each subsequent request. It's normal to use the HTTP header `Authorization: Bearer <token>`, although it can also be sent in URL parameters or request body.
5. **Verification:** Each time a request arrives with a JWT, the receiving server extracts the token, divides its three parts, decodes the header and payload, and **verifies the signature** using the same key (or corresponding public key). It also checks standard claims (e.g., that it hasn't expired `exp` and that the issuer `iss`, audience `aud`, etc. are valid). If the signature is correct and the claims are acceptable, the server trusts the payload content and allows access.
6. **Using the information:** After verification, the server can use the payload information (e.g., user ID or roles) to authorize the requested action, without needing to query the database for the user in each request.
7. **Logout/expiration:** If the user logs out or the token expires, the client simply discards the JWT. Since the token is stateless, the server doesn't need to delete anything in memory (no session to destroy); simply when an expired or invalid token arrives, access is rejected.

This flow illustrates how JWT allows delegating all necessary information to the token itself. The header and payload are signed, which guarantees their integrity and authenticity. The client only needs to forward the token; the server **doesn't maintain session state** between requests, limiting itself to signing new tokens and verifying incoming ones.

## Comparison: JWT vs Sessions and Cookies

Unlike the traditional **session with cookies** scheme, where the server creates a session record on its side and sends only an identifier (usually in a cookie) to the client, with JWT the token contains all relevant information by itself. This generates key differences:

### State Management
Classic sessions are *stateful*: the server must store data for each connected user and look up the corresponding session in each request. In contrast, JWT is *stateless*: the server doesn't store session information, as all data travels in the signed token. This makes JWT more scalable in distributed systems (any instance can validate a token without synchronizing sessions).

### Transport and Compatibility
Cookies are handled automatically by the browser and are usually limited to the same domain; JWTs can be sent in HTTP headers or parameters and work well across domains (CORS). Therefore, they're more flexible for modern APIs, mobile, and IoT environments where there's no native cookie support.

### Revocation and Invalidity
In sessions, it's easy to invalidate access by deleting the session on the server; in JWT there's no immediate "logout" mechanism. A JWT remains valid until it expires, unless an additional system (blacklist) is implemented to invalidate it prematurely. This requires additional caution (e.g., short-lived tokens and refresh tokens).

### Security and Storage
Session cookies can be marked `HttpOnly` and `Secure`, which protects them from XSS attacks but exposes them to CSRF if not configured properly. A JWT stored in, for example, localStorage runs the risk of XSS (an attacker could steal it with malicious code). To mitigate this, it's recommended to store the JWT in an `HttpOnly` cookie (protected against XSS) with `Secure` and `SameSite` attributes. In the end, neither approach is intrinsically "bad," but each has its advantages: sessions are simple and secure within the same domain, while JWT enables more distributed architectures and client mobility.

### Token Content
With sessions, the cookie only carries an ID, and all user data resides on the server. In JWT, additional data (roles, profile, etc.) can be included directly in the payload. This avoids extra database queries, but also means that nothing should be stored that shouldn't be visible to whoever possesses the token (since the payload is just Base64, not encrypted).

In summary, JWT and cookies/sessions are complementary approaches: using cookies+sessions is appropriate for traditional web applications, while JWT is usually chosen in systems with many different clients (SPAs, mobile, microservices, etc.) for its lightness and domain neutrality. (Simple "opaque" tokens, i.e., random strings without embedded information, work similarly to sessions: the server must store them in a database to know what user they represent. JWTs, on the other hand, carry the information within themselves.)

## Advantages and Disadvantages of Using JWT

JWTs offer several important **advantages** in distributed authentication and authorization:

### Advantages

- **Stateless and scalable:** By not requiring server sessions, they enable more *stateless* and scalable systems.
- **Compact and easy transport:** Being based on JSON and encoded in Base64URL, they're very compact and easy to send in URLs and HTTP headers, reducing network load.
- **Self-contained:** The token includes all necessary information (claims) to authorize the request, avoiding database queries in each request.
- **Digital signatures:** They're signed (symmetric with HMAC or asymmetric with RSA/ECDSA), which guarantees that the data hasn't been altered and comes from a trusted issuer.
- **Interoperability and SSO:** They're a widely supported standard across multiple languages and platforms, facilitating interoperability. The same JWT can be used for Single Sign-On between different applications/services.
- **Extensible:** Custom claims can be added to the payload without affecting compatibility, adapting to specific needs.

### Disadvantages

- **Impossibility of immediate revocation:** A JWT is valid until it expires. If you want to "invalidate" it before, extra mechanisms must be implemented (blacklists or token rotation), which adds complexity. This makes operations like forcing a centralized logout or revoking permissions on the fly difficult.
- **Token size:** Compared to a session ID, the JWT is usually larger. Each request must include the entire token, so a JWT with many claims can increase HTTP traffic.
- **Content exposure:** The JWT payload is not encrypted, only encoded in Base64. This means that anyone who captures the token can read its claims (non-sensitive), so confidential information should not be included without encryption.
- **XSS/CSRF vulnerability:** If stored poorly (e.g., in localStorage), JWTs can be stolen via XSS. If used in cookies, they can be subject to CSRF attacks if measures like `SameSite` aren't applied.
- **Signature strength:** If an insecure algorithm (like `alg:"none"`) is used or the secret key is leaked, any attacker could forge false tokens. It's crucial to use robust algorithms (e.g., HS256 or RS256) and protect keys well.
- **Additional complexity:** Managing JWT usually also involves a **refresh token** system to periodically renew credentials, expiration management, and secure storage, which is more complex than the simple traditional session scheme.

Overall, the benefits of JWT (self-contained, stateless, and signed) must be weighed against these risks and additional costs. Implementing good practices (short-lived tokens, strong signature, secure storage, re-authentication/refresh) is essential to take advantage of their benefits without compromising security.

## Known Security Risks and Mitigation

Among the known vulnerabilities when using JWT are:

### Security Risks

- **Unsigned tokens or weak signatures:** Using the `"alg":"none"` algorithm or weak secret keys makes the token not truly authenticated. Always require robust algorithms (e.g., HS256, RS256) and reject unsigned JWTs.
- **Sensitive information in payload:** Since the content travels encoded but not encrypted, any data in the `payload` can be read if the token is exposed. Never store passwords or other secrets directly in the payload. If it's necessary to protect data, it should be encrypted separately or use encrypted JWTs (JWE).
- **Token theft and replay attacks:** An attacker who steals a valid JWT can use it until expiration. To mitigate this, it's recommended to give them **short life** (claim `exp`) and also use issuance claims (`iat`) and validity (`nbf`). Additionally, implementing **refresh tokens** allows access tokens (short-term) to be renewed without asking for login again. Thus, even if a token is stolen, it will lose validity quickly.
- **Insecure storage:** Don't store the JWT in places vulnerable to XSS, like localStorage without precautions. The ideal is an **`HttpOnly` and `Secure` cookie**: these cookies are not accessible by JavaScript, which protects them from XSS, and being sent automatically in each HTTPS request prevents a token from being exposed on the client. In any case, always use **HTTPS** to avoid token interception on the network.
- **CSRF in cookies:** If the JWT is sent in a cookie, it must be protected against CSRF by configuring attributes like `SameSite=Strict` or `Lax`. These configurations make the browser not send the cookie in requests from other origins, mitigating CSRF attacks.
- **Key rotation:** It's good practice to periodically rotate signing keys (or key pairs in RSA) to limit the impact if a key is compromised. This must be coordinated with token expiration management.

### Mitigation Strategies

In summary, mitigation involves applying a combination of practices: **short expiration tokens**, use of **robust algorithms**, **secure storage** (e.g., HttpOnly cookies with `SameSite`), use of **refresh tokens**, and always **communication over HTTPS**. These measures minimize the risks associated with JWT while taking advantage of their benefits.

## Common Use Cases

Some typical scenarios where JWT is used in modern systems are:

### Authentication in REST APIs/SOAs
After login, each client (web, mobile, or service) includes the JWT in request headers to access protected routes. The server validates the JWT in each call, without maintaining state. This is very common in API backends and single-page applications (SPA).

### Single Sign-On (SSO) Between Domains or Services
The same JWT can be consumed by different services, allowing the user not to have to authenticate repeatedly. Companies use it so that a single login serves multiple applications.

### Authorization Between Microservices
In microservice architectures, JWT allows propagating user identity and permissions between services without retransmitting credentials or consulting a central server at each step. Each microservice can verify the token to authorize the request.

### Mobile Applications and IoT
Since JWTs don't depend on browser cookies, they're ideal for native mobile clients, desktop applications, or IoT devices that consume APIs. They're easily sent in HTTP requests (e.g., in headers) without domain issues.

### Secure Information Exchange
Any scenario where it's necessary to share data that the receiver must trust. For example, a service can issue a JWT with session data or permissions that another party consumes with confidence thanks to the signature.

In practice, **virtually any modern system** of authentication and authorization can benefit from JWT when scalability and mobility are needed. Its use is widespread in startups, public APIs (social networks, payments, etc.), cloud systems, and identity protocols (like OAuth 2.0 / OpenID Connect often use JWT for access and ID tokens). In all these cases, JWT allows exchanging **verifiable credentials** in a simple and standardized way.

## Key Takeaways

- **JWT** is a compact, self-contained way to transmit verified information
- **Three parts**: Header, Payload, and Signature (all Base64URL encoded)
- **Stateless**: No server-side session storage required
- **Signed**: Ensures integrity and authenticity
- **Flexible**: Can be sent in headers, parameters, or URLs
- **Standard**: Widely supported across platforms and languages

## Security Best Practices

1. **Use strong algorithms** (HS256, RS256) - never use `"alg":"none"`
2. **Keep tokens short-lived** with refresh mechanisms
3. **Store securely** - prefer HttpOnly cookies over localStorage
4. **Always use HTTPS** for transmission
5. **Don't store sensitive data** in the payload
6. **Implement proper key management** and rotation
7. **Validate all claims** including expiration and issuer

## Next Steps

Now that you understand JWT, you can:
- Implement JWT authentication in your APIs
- Design secure token-based systems
- Integrate with OAuth 2.0 and OpenID Connect
- Build scalable, stateless applications
- Implement proper security measures

