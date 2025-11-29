---
title: "About OAuth 2.0"
date: "2025-11-28"
summary: "📝 Quick notes on OAuth 2.0..."
tags: ["security"]
---

## Definition

OAuth 2.0 is an authorization framework that allows one application (Client) to access protected resources on another application (Resource Server) on behalf of a user (Resource Owner), without ever exposing the user's password.
> In a word, Login with Google.

## Roles

| Role | Desc |
| ---- | ---- |
| Resource Owner | The user (data owner) |
| Client | The application requesting access |
| Authorization Server | Issues tokens |
| Resource Server | Accepts tokens & returns real data |

### An Example: Login with Google

1. Resource Owner
    - The person who owns the data.
2. Client
    - The app that wants to access the user’s data.
        - A service like Discord, or a web app.
        - This app wants permission to access something from Google.
3. Authorization Server
    - The server that handles login and issues tokens.
    - Google’s OAuth server:
        - Shows you Google’s login screen
        - Checks your password
        - Asks “Do you allow this app to access your profile?”
        - Issues authorization codes & tokens
    - It doesn't store the actual data, it only handles auth.
        - Just like a security guard who verifies your identity and gives a temporary access card.
4. Resource Server
    - The Resource Server is the server that stores the actual protected data.
        - **Authorization Server** → handles login
        - **Resource Server** → actual APIs, like:
            - https://www.googleapis.com/userinfo/v2/me
            - https://gmail.googleapis.com/gmail/v1/users/me/messages
            - These APIs return data when you have a **valid access token**.
    - The API that returns user data:
        - Name
        - Email
        - Profile picture
    - It only accepts valid access tokens.
    - And it enforces scopes: A only profile scope cannot access Gmail.

## Authorization Code Flow (with PKCE)

1. Client → Authorization Server
    - Redirect the user to the login / authorization page.
2. User → Authorization Server
    - The user logs in and grants **permission**.
3. Authorization Server → Client
    - The client receives a **authorization code** in the callback URL.
4. Client → Authorization Server
    - Exchange the authorization code for an **access token** (and possibly a refresh token).
5. Client → Resource Server
    - Use the access token to call protected APIs.

## PKCE

- PKCE: Proof Key for Code Exchange
    - Pronounced “pixy”
- A security upgrade for OAuth 2.0 Authorization code flow, for:
    - Mobile apps
    - SPA (browser JS apps)
    - Desktop apps
    - Any client that cannot safely store a client secret
- It prevents authorization code theft during the redirect.

### Why is PKCE needed?
### How PKCE works?