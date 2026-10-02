# DAY-3-SB: REST API Fundamentals


## 1. API

An API allows two software systems to communicate with each other.

## 2. REST API: Representational State Transfer

REST gives us a standard way to design APIs using HTTP requests and resources.

### Characteristics of REST

#### 2.1 Client and server have separate responsibilities

Example: front end handles UI and backend handles business logic, etc.

#### 2.2 Stateless

It means each client request is independent, and the server does not maintain client session state between requests; every request contains the information necessary for the server to process it.

```text
Request 1 ──→ Server
    ↓
Process it
    ↓
Response

Request 2 ──→ Server
    ↓
Process it independently
    ↓
Response
```

#### 2.3 Cacheable

A server's response should indicate whether the response can be stored (cached) and reused for future requests.

Caching can improve performance, reduce network traffic, and reduce server load.

A cache is temporary storage used to keep data that may be needed again.

Without cache:

```text
Client
  ↓
Server
  ↓
Response

Client
  ↓
Server
  ↓
Response

Client
  ↓
Server
  ↓
Response
```

With caching:

```text
First request:

Client
  ↓
Server
  ↓
Response
  ↓
Cache


Next request:

Client
  ↓
Cache
  ↓
Response
```

**Does every REST response get cached?**

REST requires that responses indicate whether they are cacheable, allowing clients or intermediaries to reuse responses when appropriate.

### How does the response say "I can be cached"?

The server sends HTTP response headers along with the response.

One of the most important headers is `Cache-Control`.

For example, the server might respond as:

```text
HTTP/1.1 200 OK
Cache-Control: max-age=3600
Content-Type: application/json
```

```text
[
  {
    "id": 1,
    "name": "Laptop"
  }
]
```

So this tells the client: you can reuse this response for 3600 seconds (1 hour), subject to the caching rules.

### Where does cache exist?

It does not exist inside your Spring Boot application. It can exist in different places like:

- Browser cache
- CDN
- Proxy cache
- API gateway
- Other caching infrastructure

### What if the server says it should NOT be cached?

The server can send:

```text
Cache-Control: no-store
```

This tells caches not to store the response.

For example, for sensitive information, you might not want the response stored by a cache.

So:

```text
Cache-Control: max-age=3600
        ↓
Response can be cached/reused according to the rules

Cache-Control: no-store
        ↓
Do not store the response
```

> **Note:** HTTP provides caching mechanisms, and REST's cacheable constraint says that responses should indicate whether they can be cached.

The cached response becomes stale after the 1 hour, so the cache normally cannot simply reuse that old response.

Suppose the server responds:

```text
HTTP/1.1 200 OK
Cache-Control: max-age=3600
Content-Type: application/json
```

The cache stores the response and starts counting:

```text
0 min ──────────────────────── 60 min
      Fresh              Expired/Stale
```

**What happens within 1 hour?**

User requests:

```text
GET /employees
```

If the cached response is still fresh:

```text
Client
  ↓
Cache
  ↓
Cached response returned
```

The request may **not need to reach your Spring Boot server**.

**What happens after 1 hour?**

Suppose the user requests:

```text
GET /employees
```

again after 65 minutes.

The cache sees:

```text
max-age = 3600 seconds
```

and realizes: "My stored response is no longer fresh."

It cannot normally serve that stale response as a fresh response. Depending on the caching rules and headers, it may:

1. Revalidate with the server, or
2. Fetch a fresh response from the server.

```text
After 1 hour
    ↓
Cache is stale
    ↓
Revalidate
    ↓
Data changed?
    ↓           ↓
   No          Yes
    ↓           ↓
   304         200
    ↓           ↓
Reuse old    New response
response
```

`Cache-Control: max-age=3600` doesn't literally mean "delete this response after 1 hour." It means the response is considered fresh for up to 3600 seconds. After that, it's **stale**, and the cache needs to follow the applicable caching/revalidation rules before using it.

#### 2.4 Uniform Interface

Uniform Interface means REST APIs follow a consistent and standard way of communicating with resources.

Instead of every API behaving differently, REST APIs use common conventions such as:

- Resources are identified using URLs
- HTTP methods have standard meanings
- Standard HTTP status codes are used
- Data is commonly represented in formats such as JSON

This does not mean every REST API must have exactly the same URLs. It means APIs should follow **consistent, standardized principles** when exposing and interacting with resources.

#### 2.5 Layered structure

The Layered System constraint allows a REST architecture to consist of multiple intermediary layers between the client and the server, such as API gateways, load balancers, and security layers. The client does not need to know whether it is communicating directly with the final server or through intermediaries.

You don't want the client to know:

```text
Server 1
Server 2
Server 3
...
Server 10
```

Instead:

```text
                   ┌── Server 1
                   │
Client → Load Balancer ── Server 2
                   │
                   └── Server 3
```

The client only communicates with the **Load Balancer**.

The Load Balancer decides which backend server should receive the request.

### How does this help?

- **Security:** You can put authentication/security layers between the client and application.
- **Scalability:** A load balancer can distribute requests across multiple servers.
- **Separation of responsibilities:** Different layers can handle different responsibilities.
- **Maintainability:** You can change internal layers without necessarily changing how the client communicates with the API.

---

## 3. HTTP

HTTP is a communication protocol used for transferring data between a client and a server over a network, especially the web.

A request can have things like:

- HTTP Method
- URL
- Headers
- Request Body

| HTTP Method | Typical purpose |
|---|---|
| GET | Retrieve data |
| POST | Create data |
| PUT | Update/replace data |
| DELETE | Delete data |
