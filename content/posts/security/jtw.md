---
title: "Learning JWT"
date: "2025-11-28"
summary: "📝 Quick notes on learning JWT..."
tags: ["security"]
---

## Definition

- JWT: JSON Web Token
- A signed string used to prove “who you are”.

## Structure

```
header.payload.signature
```

1. Header: Algorithm, token type

```
{
  "alg": "HS256",
  "typ": "JWT"
}

```

2. Payload: User data, claims

```
{
  "sub": "12345",
  "name": "Fū",
  "role": "admin",
  "exp": 1732760400
}
```


3. Signature: To prevent tampering

```
HMACSHA256(
    base64url(header) + "." + base64url(payload),
    secret_key
)
```

## Work Flow

1. User login, backend checks credentials

```
POST /login
{
  "email": "test@example.com",
  "password": "1234"
}

```
2. Backend creates a JWT with user info

```
{
  "token": "xxxxx.yyyyy.zzzzz"
}
```

3. Backend sends the JWT to the client
4. Client stores it (cookie or localStorage)
5. Client sends JWT in every request

```
Authorization: Bearer xxxxx.yyyyy.zzzzz
```

6. Backend verifies signature, allow or deny request

## Pros and Cons?

- Pros
    - No session storage required (server is stateless)
        - Especially works well in microservices
    - Fast, simple
    - A standard, widely adopted
- Cons
    - Cannot be revoked easily until expiration
    - Payload is visible, not encrypted