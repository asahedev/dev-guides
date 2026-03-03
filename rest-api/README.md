# 🌐 REST API Guide for Beginners

> **Author:** [@asahedev](https://github.com/asahedev)  
> **Format:** Reads as a guide, doubles as a cheat sheet  
> **Language examples:** Python 🐍

---

## Table of Contents

1. [What is an API?](#1-what-is-an-api)
2. [What is REST?](#2-what-is-rest)
3. [HTTP Methods](#3-http-methods)
4. [Status Codes](#4-status-codes)
5. [Authentication & API Keys](#5-authentication--api-keys)
6. [Query Parameters & Filtering](#6-query-parameters--filtering)
7. [Python Examples](#7-python-examples)
8. [REST Best Practices](#8-rest-best-practices)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. What is an API?

An **API (Application Programming Interface)** is a messenger that lets two pieces of software talk to each other.

Think of it like a waiter in a restaurant:
- **You** (your app) place an order
- **The waiter** (the API) takes your request to the kitchen
- **The kitchen** (the server/database) prepares the response
- The waiter brings it back to you

**Real example:** When you click "Sign in with Google" on a website, that site is using Google's API to ask: *"Is this person who they say they are?"*

---

## 2. What is REST?

**REST (Representational State Transfer)** is a set of rules for designing APIs in a clean, predictable way. An API that follows these rules is called a **RESTful API**.

The core ideas are simple:

- **Everything is a resource** — users, products, orders — each gets its own URL
- **Use HTTP methods correctly** — the URL describes *what*, the method describes *the action*
- **Stateless** — every request must contain all the info needed; the server remembers nothing between requests
- **Responses are structured** — data is usually returned as JSON, though REST doesn't strictly require it

---

## 3. HTTP Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `GET` | Fetch/read data | Get all users |
| `POST` | Create new data | Add a new user |
| `PUT` | Replace an entire resource | Replace all of user 5's data |
| `PATCH` | Partially update a resource | Update only user 5's email |
| `DELETE` | Remove data | Delete a user |

> 💡 **PUT vs PATCH:** Use `PUT` when replacing the whole resource, `PATCH` when updating only specific fields.

### RESTful URL patterns

```
GET    /users           → get ALL users
GET    /users/5         → get user with ID 5
POST   /users           → create a new user
PUT    /users/5         → update user with ID 5
DELETE /users/5         → delete user with ID 5
```

> 💡 **Rule:** The URL describes the **resource**, the HTTP method describes the **action**.

---

## 4. Status Codes

The server always replies with a **status code** telling you what happened.

### Success (2xx)

| Code | Meaning | When you'll see it |
|------|---------|-------------------|
| `200` | OK | Successful GET or PUT |
| `201` | Created | Successful POST |
| `204` | No Content | Successful DELETE |

### Client Errors (4xx) — *your fault*

| Code | Meaning | When you'll see it |
|------|---------|-------------------|
| `400` | Bad Request | Malformed JSON or missing fields |
| `401` | Unauthorized | Missing or invalid API key |
| `403` | Forbidden | Valid key but no permission |
| `404` | Not Found | Resource doesn't exist |
| `422` | Unprocessable | Valid request but failed validation |

### Server Errors (5xx) — *their fault*

| Code | Meaning | When you'll see it |
|------|---------|-------------------|
| `500` | Internal Server Error | Something crashed on the server |
| `503` | Service Unavailable | Server is down or overloaded |

---

## 5. Authentication & API Keys

Most real APIs require you to prove who you are. The most common ways:

### API Key (simplest)
A secret token that identifies your app. **Never share it publicly or hardcode it in your code.**

```python
import os

api_key = os.environ.get("MY_API_KEY")  # ✅ load from environment variable
```

### Sending your API key

**Option 1 — In the header (most common & secure):**
```python
import requests

headers = {"Authorization": f"Bearer {api_key}"}
response = requests.get("https://api.example.com/data", headers=headers)
```

**Option 2 — As a query parameter (less secure, avoid for sensitive keys):**
```python
import requests

response = requests.get("https://api.example.com/data", params={"api_key": api_key})
```

> ⚠️ **Never put API keys in URLs for sensitive APIs** — URLs get logged by servers and browsers.

---

## 6. Query Parameters & Filtering

Use `?key=value` in the URL to filter, sort, or paginate results — don't create separate endpoints for each use case.

```
GET /products?category=electronics
GET /products?sort=price&order=asc
GET /products?category=electronics&sort=price&page=2
```

In Python:

```python
response = requests.get(
    "https://api.store.com/products",
    params={
        "category": "electronics",
        "sort": "price",
        "order": "asc",
        "page": 2
    }
)
```

The `requests` library automatically builds the URL for you — clean and readable.

---

## 7. Python Examples

### Setup
```bash
pip install requests
```

### GET — Fetch data
```python
import requests

response = requests.get("https://api.example.com/users")

if response.status_code == 200:
    users = response.json()
    print(f"Found {len(users)} users")
else:
    print(f"Error: {response.status_code}")
```

### POST — Create data
```python
new_user = {
    "name": "Alice",
    "email": "alice@example.com"
}

response = requests.post(
    "https://api.example.com/users",
    json=new_user,
    headers={"Authorization": "Bearer YOUR_API_KEY"}
)

if response.status_code == 201:
    print("User created!", response.json())
```

### PUT — Update data
```python
updated_data = {"email": "newemail@example.com"}

response = requests.put(
    "https://api.example.com/users/5",
    json=updated_data,
    headers={"Authorization": "Bearer YOUR_API_KEY"}
)
```

### PATCH — Partially update data
```python
# Only update the email — leave everything else unchanged
partial_update = {"email": "updatedemail@example.com"}

response = requests.patch(
    "https://api.example.com/users/5",
    json=partial_update,
    headers={"Authorization": "Bearer YOUR_API_KEY"}
)

if response.status_code == 200:
    print("User partially updated!", response.json())
```

### DELETE — Remove data
```python
response = requests.delete(
    "https://api.example.com/users/5",
    headers={"Authorization": "Bearer YOUR_API_KEY"}
)

if response.status_code == 204:
    print("User deleted successfully")
```

### Full example with error handling
```python
import requests
import os

BASE_URL = "https://api.example.com"
HEADERS = {"Authorization": f"Bearer {os.environ.get('API_KEY')}"}

def get_user(user_id):
    response = requests.get(f"{BASE_URL}/users/{user_id}", headers=HEADERS)

    if response.status_code == 200:
        return response.json()
    elif response.status_code == 404:
        print(f"User {user_id} not found")
    elif response.status_code == 401:
        print("Invalid API key")
    else:
        print(f"Unexpected error: {response.status_code}")

    return None
```

---

## 8. REST Best Practices

### ✅ Use plural nouns for endpoints — never verbs
```
✅ /users
✅ /products
❌ /getUsers       ← the HTTP method already says "get"
❌ /deleteProduct  ← use DELETE /products/5 instead
```

### ✅ Nest resources logically
```
GET /users/5/orders        → all orders for user 5
GET /users/5/orders/12     → order 12 belonging to user 5
```

### ✅ Always check the status code
```python
# ❌ Don't assume success
data = response.json()

# ✅ Always verify first
if response.status_code == 200:
    data = response.json()
```

### ✅ Use environment variables for secrets
```python
# ❌ Never do this
api_key = "abc123supersecret"

# ✅ Always do this
import os
api_key = os.environ.get("API_KEY")
```

### ✅ Use query parameters for filtering & sorting
```
✅ GET /products?category=shoes&sort=price
❌ GET /products/shoes/sorted-by-price
```

---

## 9. Common Pitfalls

| Pitfall | Why it's wrong | Fix |
|--------|---------------|-----|
| Using GET to modify data | GET should never change server state | Use POST / PUT / DELETE |
| Putting passwords in URLs | URLs get logged and can be exposed | Use request body or headers |
| Hardcoding API keys in code | Anyone reading your code can steal them | Use environment variables |
| Not handling errors | Your app will crash unexpectedly | Always check `status_code` |
| Using verbs in endpoints | Breaks REST conventions | Use nouns + HTTP methods |
| Ignoring 4xx vs 5xx | Different problems need different fixes | Handle each category separately |

---

## 10. Quick Reference Cheat Sheet

```
# HTTP Methods
GET    → Read       → 200 OK
POST   → Create     → 201 Created
PUT    → Update     → 200 OK
DELETE → Delete     → 204 No Content

# Common Status Codes
200 OK | 201 Created | 204 No Content
400 Bad Request | 401 Unauthorized | 403 Forbidden | 404 Not Found
500 Internal Server Error

# Python one-liner examples
requests.get(url, headers=headers)
requests.post(url, json=data, headers=headers)
requests.put(url, json=data, headers=headers)
requests.delete(url, headers=headers)
```

---

*Found this helpful? Give it a ⭐ and share it with others!*
