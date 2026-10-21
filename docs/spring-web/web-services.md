# Web Services

## WebServices

[https://blog.postman.com/soap-vs-rest/](https://blog.postman.com/soap-vs-rest/)

## SOAP vs REST

|  | SOAP | REST |
| :---- | :---- | :---- |
| Stands for  | Simple Object Access Protocol | Representational State Transfer |
| What is it? | SOAP is a protocol for communication between applications | REST is an architecture style for designing communication interfaces. |
| Design | SOAP API exposes the operation. | REST API exposes the data. |
| Transport Protocol | SOAP is independent and can work with any transport protocol. | REST works only with HTTPS. |
| Data format | SOAP supports only XML data exchange. | REST supports XML, JSON, plain text, HTML. |
| Performance | SOAP messages are larger, which makes communication slower. | REST has faster performance due to smaller messages and caching support. |
| Scalability | SOAP is difficult to scale. The server maintains state by storing all previous messages exchanged with a client. | REST is easy to scale. It’s stateless, so every message is processed independently of previous messages. |
| Security | SOAP supports encryption with additional overheads. | REST supports encryption without affecting performance. |
| Use case | SOAP is useful in legacy applications and private APIs. | REST is useful in modern applications and public APIs. |

### Use Cases

**When to Use SOAP**

1. **Developing private APIs, especially for large enterprises.**
   SOAP allows data to be transferred in a decentralized, distributed environment. It also has lots of web security mechanisms. These qualities make it ideal for enterprise solutions.

2. **Working with stateful operations.**
   Unlike calls to REST APIs, calls to SOAP APIs are stateful. This means the server stores information about the client and uses that information over a series of requests or chain of operations.
   While this requires more server resources and bandwidth, it’s important if performing repetitive or chained tasks, like bank transfers.

3. **Using an underlying transport protocol other than HTTP.**
   SOAP is independent of an underlying transport protocol, so you don’t have to use HTTP. Instead, you could use SMTP (Simple Mail Transfer Protocol), JMS (Java Messaging Service), or another transport protocol, depending on your application.

**When to Use REST**

1. **Developing public APIs.**
   Many consider REST APIs easier to use and adopt than SOAP APIs. which makes them ideal for creating public web services. REST also lacks some built-in security features that SOAP has — but they aren’t necessary when working with public data and services.

2. **Working with limited server resources and bandwidth.**
   All calls to a REST API must be stateless. This means that every interaction is independent so each request and response provides all the information required to complete that interaction. Since the server interprets every request as brand new, the server does not store information on past requests.
   This greatly reduces the amount of server memory needed. It improves performance since the server is not required to take additional action or retrieve past data when fulfilling a request.
   Because REST is stateless, data can be cached, which also saves server resources and bandwidth.
   Finally, REST APIs can use different data formats, like JSON, which is lighter than XML. This makes them faster and more efficient than most SOAP APIs.

3. **Building mobile applications.**
   Because REST is lightweight, efficient, stateless, and cacheable, it’s ideal for building mobile applications.

### Alternatives

1. **JSON**
   JSON (JavaScript object notation) is an open standard file format used to transmit data objects between many applications. It is a lightweight format to store and transfer data and is often used when sending data from a server to a web page. The simplicity and speedy transmission of JSON make it a viable alternative in many situations.

2. **gRPC**
   gRPC (remote procedure call or RPC) is an open-source system developed by Google that uses [HTTP/2](https://www.upwork.com/resources/what-is-http2). It is commonly used to provide a connection to different applications in a microservices architecture, and it allows mobile devices to communicate with backend services.
   The advantages of gRPC include more lightweight messages than JSON, high performance, built-in code generation, and support for more connection options such as streaming data.

3. **Falcor**
   Developed by Netflix, [Falcor](https://netflix.github.io/falcor/) is a JavaScript library that assists in data fetching. It allows you to retrieve data from different sources and combine it into a single model. As a result, you can more easily pass data through different UI components and display it to users.
   Falcor also supports client-side caching, meaning you can retrieve values from the local storage instead of making random network calls—which can otherwise be time-consuming and resource-intensive.

4. **GraphQL**
   GraphQL is a query language used to efficiently load data from a server to a client. Created by Facebook, this relatively new technology supports reading, writing, and subscribing to changes to data, and GraphQL servers are available for languages like JavaScript, Python, C++, and more.
   Just like REST, GraphQL communicates using HTTP and uses the JSON data format. One of the key differences and benefits is the possibility to specify the data you want to be returned from the server in one API call.
   For example, if we want to fetch a customer, a customer order, and the order’s shipment status using REST, we would have to conduct separate HTTP requests for each piece of data. With GraphQL, we can fetch everything using one request, which eliminates the HTTP overhead for each call.

## Authentication vs Authorization

[https://www.postman.com/api-platform/api-authentication/](https://www.postman.com/api-platform/api-authentication/)

| Authentication | Authorization |
| :---- | :---- |
| In the [authentication](https://www.geeksforgeeks.org/authentication-in-computer-network/) process, the identity of users are checked for providing access to the system. | While in [authorization](https://www.geeksforgeeks.org/what-is-aaa-authentication-authorization-and-accounting/) process, the person’s or user’s authorities are checked for accessing the resources. |
| In the authentication process, users or persons are verified. | While in this process, users or persons are validated. |
| It is done before the authorization process. | While this process is done after the authentication process. |
| It needs usually the user’s login details. | While it needs the user’s privilege or security levels. |
| Authentication determines whether the person is user or not. | While it determines What permission does the user have? |
| Generally, transmit information through an ID Token. | Generally, transmit information through an Access Token. |
| The OpenID Connect (OIDC) protocol is an authentication protocol that is generally in charge of user authentication process.  | The OAuth 2.0 protocol governs the overall system of user authorization process. |
| Popular Authentication Techniques- Password-Based Authentication Passwordless Authentication 2FA/MFA (Two-Factor Authentication / Multi-Factor Authentication) [Single sign-on (SSO)](https://www.geeksforgeeks.org/introduction-of-single-sign-on-sso/) Social authentication | Popular  Authorization Techniques- Role-Based Access Controls (RBAC) [JSON web token (JWT) Authorization](https://www.geeksforgeeks.org/json-web-token-jwt/) SAML Authorization OpenID Authorization OAuth 2.0 Authorization |
| The authentication credentials can be changed in part as and when required by the user. | The authorization permissions cannot be changed by user as these are granted by the owner of the system and only he/she has the access to change it. |
| The user authentication is visible at user end. | The user authorization is not visible at the user end. |
| The user authentication is identified with username, password, face recognition, retina scan, fingerprints, etc.  | The user authorization is carried out through the access rights to resources by using roles that have been pre-defined. |
| Example: Employees in a company are required to authenticate through the network before accessing their company email. | Example: After an employee successfully authenticates, the system determines what information the employees are allowed to access.  |

---

## GraphQL

**GraphQL** — Query language for APIs; clients request exactly the fields they need from a single endpoint.

### Core Concepts

- **Single Endpoint**: Usually POST /graphql (vs REST with multiple URLs)
- **Strongly Typed Schema**: Defines all possible queries, mutations, and subscriptions
- **No Over/Under-fetching**: Get exactly what you need in one request
- **Resolvers**: Functions that fetch data for each field

### Query vs Mutation vs Subscription

**Query** (read):
```graphql
query {
  user(id: 123) {
    name
    email
    posts {
      id
      title
    }
  }
}
```

**Mutation** (write):
```graphql
mutation {
  createUser(input: {name: "John", email: "john@example.com"}) {
    id
    name
  }
}
```

**Subscription** (real-time):
```graphql
subscription {
  userCreated {
    id
    name
  }
}
```

### Advantages Over REST

- **One Request**: Fetch user + posts + comments in single call (REST needs 3 requests)
- **Flexible**: Frontend chooses fields; backend doesn't need separate endpoints
- **Strongly Typed**: Schema serves as documentation and IDE autocomplete
- **Real-time**: Built-in subscriptions for live updates

### Disadvantages

- **Complexity**: Steeper learning curve, complex queries hard to optimize
- **Caching**: HTTP caching works better with REST; GraphQL needs custom caching
- **Monitoring**: Harder to track errors and performance (all POST to same endpoint)
- **File Upload**: Not as straightforward as REST multipart forms

### When to Use GraphQL

✅ **Use if:**
- Complex/nested data fetching
- Multiple client types (web, mobile, IoT) with different data needs
- Real-time data important (subscriptions)
- Rapid frontend iteration

❌ **Avoid if:**
- Simple CRUD operations (REST is simpler)
- File uploads important
- Team unfamiliar with GraphQL
- Real-time not needed

### Spring Boot GraphQL Integration

```java
// Dependency
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>

// schema.graphqls
type Query {
  user(id: ID!): User
  users: [User!]!
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
}

input CreateUserInput {
  name: String!
  email: String!
}

// Controller
@Controller
public class UserGraphQL {
    private UserService userService;
    private PostService postService;
    
    @QueryMapping
    public User user(@Argument Long id) {
        return userService.findById(id);
    }
    
    @QueryMapping
    public List<User> users() {
        return userService.findAll();
    }
    
    @MutationMapping
    public User createUser(@Argument CreateUserInput input) {
        return userService.create(input);
    }
    
    @SchemaMapping
    public List<Post> posts(User user) {
        return postService.findByAuthor(user);
    }
}

// Client request
POST /graphql
{
  "query": "{ user(id: 1) { name email posts { title } } }"
}

// Response
{
  "data": {
    "user": {
      "name": "John",
      "email": "john@example.com",
      "posts": [
        {"title": "GraphQL Intro"},
        {"title": "REST vs GraphQL"}
      ]
    }
  }
}
```

---

## API Authentication & Authorization Methods

### Authentication Mechanisms

#### 1. Basic Authentication
Username + password in Authorization header (base64 encoded):
```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

- ✅ Simple, no session needed
- ❌ Credentials exposed if not over HTTPS
- ❌ Can't log out (always re-authenticate)
- **Use**: Simple APIs, internal services

#### 2. API Keys
Secret token passed in header or query param:
```
Authorization: Bearer sk_live_123456789
X-API-Key: sk_live_123456789
```

- ✅ Simple to implement and use
- ❌ No standard format
- ❌ If leaked, entire API compromised
- **Use**: Public APIs, third-party integrations

#### 3. JWT (JSON Web Tokens)
Stateless token containing claims (user, roles, expiry):
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

Token structure: `header.payload.signature`
- ✅ Stateless, scales well
- ✅ Can include roles/permissions
- ✅ Works across services
- ❌ Can't revoke until expiry (need blacklist)
- **Use**: SPAs, microservices, mobile apps

#### 4. OAuth 2.0
Delegated authorization (user doesn't share password):
```
Authorization: Bearer <access_token>
```

Flow:
1. User redirects to OAuth provider
2. User grants permission
3. App receives auth code
4. App exchanges code for access token
5. App uses token to access API

- ✅ User password never shared with app
- ✅ User can revoke access anytime
- ✅ Standardized protocol
- ❌ More complex setup
- **Use**: Third-party integrations, social login

#### 5. OpenID Connect (OIDC)
OAuth 2.0 + authentication (adds ID token with user info):
```json
{
  "access_token": "...",
  "id_token": "eyJ...",  // User info
  "token_type": "Bearer"
}
```

- ✅ Combines auth + authorization
- ✅ User identity verified
- ❌ More complex
- **Use**: Enterprise SSO, third-party identity

#### 6. Mutual TLS (mTLS)
Certificate-based authentication for service-to-service:
```
Client certificate → Server validates → Mutual trust
```

- ✅ Strongest security (client + server certs)
- ✅ No passwords
- ❌ Complex certificate management
- **Use**: Microservices, internal APIs

#### 7. Session-Based (Cookies)
Server creates session, stores in memory/DB, returns session ID in cookie:
```
Set-Cookie: JSESSIONID=abc123; HttpOnly; Secure
```

- ✅ Built-in to web frameworks
- ✅ Familiar to developers
- ❌ Requires server-side storage
- ❌ Doesn't scale across servers (needs session replication)
- **Use**: Traditional monolithic apps

### Comparison Table

| Method | Stateless | Scalable | Secure | Use Case |
|--------|-----------|----------|--------|----------|
| **Basic Auth** | ✅ | ✅ | ❌ | Internal APIs only |
| **API Keys** | ✅ | ✅ | ⚠️ | Public APIs, third-party |
| **JWT** | ✅ | ✅ | ✅ | Modern SPAs, microservices |
| **OAuth 2.0** | ✅ | ✅ | ✅ | Third-party integrations |
| **OIDC** | ✅ | ✅ | ✅ | Enterprise SSO |
| **mTLS** | ✅ | ✅ | ✅✅ | Service-to-service |
| **Session** | ❌ | ❌ | ✅ | Monolithic apps |

### Single Sign-On (SSO)

**SSO** — User logs in once, accesses multiple applications without re-authentication.

#### How SSO Works

1. User visits App A
2. App A redirects to SSO Server (e.g., Okta, Azure AD, Keycloak)
3. User logs in with identity provider
4. SSO Server redirects back to App A with token
5. User now authenticated in App A
6. User visits App B
7. App B redirects to same SSO Server
8. SSO Server recognizes user (already logged in)
9. App B receives token without user re-authenticating

#### Common Protocols

- **SAML 2.0**: XML-based, enterprise standard
- **OpenID Connect**: Modern, JWT-based, uses OAuth 2.0
- **Kerberos**: Unix/Windows domain authentication

#### Spring Boot SSO Example (OAuth 2.0)

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          okta:
            client-id: your-client-id
            client-secret: your-client-secret
            scope: openid,profile,email
            redirect-uri: http://localhost:8080/login/oauth2/code/okta
        provider:
          okta:
            issuer-uri: https://your-okta-domain.okta.com
            user-name-attribute: name
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .defaultSuccessUrl("/home");
        return http.build();
    }
}
```

### Authorization Strategies

#### Role-Based Access Control (RBAC)
Users have roles, roles have permissions:
```
User → Roles (ADMIN, USER) → Permissions (read, write, delete)
```

#### Attribute-Based Access Control (ABAC)
Fine-grained policies based on attributes:
```
if (user.role == "ADMIN" AND resource.classification == "PUBLIC") → allow
if (user.department == resource.owner_dept AND time.is_business_hours()) → allow
```

#### OAuth 2.0 Scopes
Request specific permissions:
```
scope=read:users,write:posts
→ Can read users but only write posts (not other resources)
```

---

**See also:** [Web Fundamentals - Security](../foundations/web-fundamentals.md), [Spring Security & RBAC](security.md), [JWT & OAuth2](jwt-oauth.md)
