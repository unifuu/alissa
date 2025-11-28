---
title: "Learning OAuth 2.0"
date: "2025-11-28"
summary: "📝 Quick notes on learning OAuth 2.0..."
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

### An Analogy: 家族を代理人として役所に行かせる

本人が役所に行けないので、家族に戸籍謄本や住民票などを代理でお願いする。

1. Resource Owner = 本人
    - 役所にある「本人の情報」の持ち主
2. Client = 代理で行く家族
    - 代理人は本人になりすまして身分証やパスワードを勝手に使ってはいけない
    - そのため、「正式な委任状」が必要になる
3. Authorization Server = 受付窓口
    - 家族が代理手続きする前に、役所の窓口で
        - 本人の意思確認
        - 委任状のチェック
        - 身分証の確認
    - 本人が許可したと確認したので、代理人に手続きの許可を発行する
4. Resource Server = 実際の住民データ・戸籍データを持っている部署
    - 下記の情報を確認する
        - 正式な書類(Access Token)があるか
        - 何の業務(Scope)が許可されているか
        - 有効期限内か

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

### An Analogy: 家族を代理人として役所に行かせる

1. Client → Authorization Server
    - 家族が本人の代わりに手続きをしたい
2. Authorization Server → Resource Owner
    - ログイン & 同意画面
        - 本当に代理して良いですか
        - 本人の身分証やハンコなど
        - 委任状の署名
3. Authorization Server → Client
    - 役所が代理人に「委任状(Authorization Code)」を発行
4. Client → Authorization Server
    - 代理人は別の窓口で「委任状(Authorization Code)」を正式な「許可証(Access Token)」に交換
5. Client → Resource Server
    - 許可証(Access Token)」が有効なら、書類を発行する

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