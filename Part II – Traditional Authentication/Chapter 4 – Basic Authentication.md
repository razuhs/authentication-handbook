# Chapter 4 – Basic Authentication

> **"Basic Authentication is one of the simplest HTTP authentication mechanisms. It sends the user's credentials with every request. Although easy to implement, it should never be used over plain HTTP because the credentials can be easily recovered."**

---

# Learning Objectives

After completing this chapter, you will be able to:

- Understand what Basic Authentication is.
- Explain how Basic Authentication works.
- Understand the structure of the `Authorization` header.
- Explain why Base64 is not encryption.
- Understand why HTTPS is mandatory.
- Identify the limitations of Basic Authentication.
- Explain why modern applications prefer Sessions or JWT over Basic Authentication.

---

# Prerequisites

Before reading this chapter, you should understand:

- HTTP Request & Response
- Authentication vs Authorization

---

# 1. Why This Chapter Exists

Imagine you're building a REST API.

The client needs to prove its identity.

One simple approach is:

> "Send your username and password with every request."

This idea became **Basic Authentication**.

It is simple.

It requires no session.

It requires no JWT.

It requires no cookies.

However, it comes with important security considerations that every backend developer must understand.

---

# 2. What is Basic Authentication?

Basic Authentication is an HTTP Authentication Scheme defined by **RFC 7617**.

The client authenticates itself by sending:

- Username
- Password

inside the HTTP `Authorization` header.

Example:

```http
Authorization: Basic YWRtaW46YWRtaW4=
```

The word **Basic** identifies the authentication scheme.

The remaining value is the Base64 encoding of:

```
username:password
```

---

# 3. How Basic Authentication Works

Suppose the user enters:

```
Username

admin

Password

admin123
```

The client combines them:

```
admin:admin123
```

Then encodes the string using Base64.

```
admin:admin123

↓

YWRtaW46YWRtaW4xMjM=
```

The request becomes:

```http
GET /api/products HTTP/1.1

Authorization: Basic YWRtaW46YWRtaW4xMjM=
```

---

# 4. Authentication Flow

```mermaid
sequenceDiagram

participant Client

participant Server

Client->>Client: username + ":" + password

Client->>Client: Base64 Encode

Client->>Server: Authorization: Basic <Base64>

Server->>Server: Decode Base64

Server->>Server: Verify Username & Password

Server-->>Client: 200 OK / 401 Unauthorized
```

---

# 5. Request Example

Suppose the username is:

```
admin
```

Password:

```
admin123
```

The HTTP request becomes:

```http
GET /api/products HTTP/1.1
Host: api.company.com

Authorization: Basic YWRtaW46YWRtaW4xMjM=
```

Notice:

The username and password are sent with **every request**.

---

# 6. Server Processing

When the server receives:

```http
Authorization: Basic YWRtaW46YWRtaW4xMjM=
```

it performs the following steps.

```text
Receive Request

↓

Extract Authorization Header

↓

Remove "Basic"

↓

Base64 Decode

↓

admin:admin123

↓

Split Username & Password

↓

Verify Against Database

↓

Authentication Result
```

If verification succeeds:

```
Authenticated
```

Otherwise:

```
401 Unauthorized
```

---

# 7. Base64 Is NOT Encryption

This is one of the biggest misconceptions among developers.

Many people believe:

> "The password is encoded, so it is secure."

This is incorrect.

Base64 is simply an **encoding algorithm**.

It is designed to convert binary data into printable ASCII characters.

Anyone can decode it.

Example:

```
YWRtaW46YWRtaW4xMjM=

↓

admin:admin123
```

No secret key is required.

No password cracking is required.

Anyone who intercepts the request can recover the credentials instantly.

---

# 8. Why HTTPS Is Mandatory

Suppose Basic Authentication is used over plain HTTP.

An attacker monitoring the network captures:

```http
Authorization: Basic YWRtaW46YWRtaW4xMjM=
```

The attacker simply decodes it.

```
admin:admin123
```

Game over.

The attacker now knows the user's credentials.

Now consider HTTPS.

```text
Browser

↓

TLS Encryption

↓

Server
```

The entire HTTP request is encrypted.

This includes:

- URL
- Headers
- Body
- Authorization Header

The attacker may capture encrypted packets,

but cannot read the credentials.

Therefore:

> **Basic Authentication should only be used over HTTPS.**

---

# 9. No Server-Side Session

Unlike Session Authentication,

Basic Authentication does not create a session.

Every request is independent.

Example:

```
Request 1

↓

Username + Password
```

```
Request 2

↓

Username + Password
```

```
Request 3

↓

Username + Password
```

The server verifies the credentials every time.

No login state is maintained.

---

# 10. Advantages

Basic Authentication has several benefits.

- Very easy to implement.
- Supported by almost every HTTP client.
- Stateless.
- No Session Management.
- No Token Management.
- Useful for simple internal APIs.

---

# 11. Limitations

Although simple, Basic Authentication has significant drawbacks.

## Credentials Sent on Every Request

Every request contains:

```
Username

Password
```

Even though HTTPS encrypts the transmission, repeatedly sending credentials increases exposure if the endpoint or client is compromised.

---

## No Logout Mechanism

The server does not maintain login state.

There is no session to destroy.

Logging out usually means:

- Closing the browser
- Clearing stored credentials
- Removing cached authentication

---

## No Token Expiration

Unlike JWT:

```
Access Token

↓

15 Minutes
```

Basic Authentication credentials remain valid until:

- Password changes
- Account disabled

---

## Password Exposure Risk

If HTTPS is not used,

credentials can be stolen immediately.

---

## Poor User Experience

The client must send credentials repeatedly.

Modern applications prefer:

- Session Authentication
- JWT Authentication

because they avoid repeatedly transmitting passwords.

---

# 12. Odoo Example

The standard Odoo web application **does not use Basic Authentication** for user login.

Instead, Odoo uses:

- Username & Password
- Session Authentication
- Session Cookie

However,

Basic Authentication may still be encountered when integrating with:

- Reverse Proxies
- Legacy APIs
- Third-party services

---

# 13. Behind the Scenes

Many developers imagine this process:

```
Username

↓

Encrypted

↓

Server
```

Reality:

```
Username

↓

Base64 Encoding

↓

Plain Text Recoverable

↓

Server
```

Base64 changes the representation.

It does **not** protect the data.

Only HTTPS provides confidentiality during transmission.

---

# 14. Security Considerations

Basic Authentication should always be combined with:

- HTTPS
- Strong Password Policy
- Rate Limiting
- Account Lockout
- Multi-Factor Authentication (when possible)

Never expose Basic Authentication over plain HTTP.

---

# 15. Common Misconceptions

### ❌ Base64 encrypts the password.

✅ Base64 only encodes the password.

Anyone can decode it.

---

### ❌ HTTPS makes Basic Authentication unnecessary.

✅ HTTPS protects the transmission.

Basic Authentication still determines **how credentials are sent**.

---

### ❌ Basic Authentication creates a session.

✅ No session is created.

Every request contains credentials.

---

### ❌ Credentials are sent only during login.

✅ Credentials are sent with every authenticated request.

---

# 16. Interview Questions

## Q1. What is Basic Authentication?

**Answer**

Basic Authentication is an HTTP authentication scheme where the client sends a Base64-encoded `username:password` string in the `Authorization` header. The server decodes the value and verifies the credentials.

---

## Q2. Is Base64 encryption?

**Answer**

No.

Base64 is an encoding algorithm.

It is completely reversible without any secret key.

---

## Q3. Why should Basic Authentication always use HTTPS?

**Answer**

Because Base64 does not protect credentials.

HTTPS encrypts the entire HTTP request, preventing attackers from reading the `Authorization` header during transmission.

---

## Q4. Does Basic Authentication create a Session?

**Answer**

No.

The client sends credentials with every request.

The server authenticates each request independently.

---

## Q5. Why is Basic Authentication uncommon in modern web applications?

**Answer**

Because:

- Credentials are sent with every request.
- There is no session management.
- There is no token expiration.
- It provides a poorer user experience than Session Authentication or JWT.

---

# 17. Summary

In this chapter, you learned:

- Basic Authentication is an HTTP authentication scheme.
- Credentials are sent in the `Authorization` header.
- The credentials are Base64 encoded, not encrypted.
- HTTPS is mandatory because Base64 provides no confidentiality.
- The server authenticates every request independently.
- Basic Authentication is simple but has significant limitations, making Session Authentication and JWT better choices for most modern applications.

---

# What's Next

Basic Authentication improved nothing about password exposure except standardizing how credentials are sent.

The next authentication mechanism attempts to solve one important problem:

> **Can we avoid sending the actual password over the network?**

In the next chapter, we will study **Digest Authentication**, explore concepts such as **Nonce**, **Realm**, **MD5 hashing**, and understand why Digest Authentication is more secure than Basic Authentication but has largely been replaced by modern authentication methods.