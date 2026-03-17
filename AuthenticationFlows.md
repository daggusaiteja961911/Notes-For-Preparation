# Authentication Flows - Comprehensive Notes

## 📋 Video Overview
- Topic: Different authentication flows developers should know
- Covers: Credential-based, session/token-based, and federated/delegated flows
- Also covers: Password storage best practices and single sign-on

---

## 🔑 Key Misconceptions Cleared

| Misconception | Truth |
|--------------|-------|
| MD5/SHA1 hashing is secure for passwords | These are fast, general-purpose algorithms - attackers can run billions of guesses/second |
| JWT is authentication | JWT is a **token format**, not authentication itself |
| HTTPS means my API is secured | HTTPS secures data in transit only; backend security determines actual access control |
| Authentication = Authorization | **Authentication** = who you are; **Authorization** = what you can do |
| 401 and 403 are the same | **401** = Unauthorized (invalid credentials); **403** = Forbidden (authenticated but no permission) |

---

## 🎯 Authentication vs Authorization

### Authentication
- Answers: "Who are you?"
- Verifies identity
- First gate before any access
- Can return **401 Unauthorized** if fails

### Authorization
- Answers: "What can you do?"
- Comes after authentication
- Determines permissions and access levels
- Can return **403 Forbidden** if access denied

**Flow**: API Request → Authentication Service → Authorization Service → Protected Resource

---

## 🔐 Password Storage Evolution

### Timeline
1. **1990s: Plain Text Passwords**
   - ❌ One breach = all passwords exposed

2. **MD5 and SHA1 Hashing**
   - ⚠️ Better but too fast (attackers can run billions of guesses/second)

3. **Bcrypt (Current Standard)**
   - ✅ Deliberately slow
   - ✅ Configurable cost factor (scales with hardware)
   - ✅ Automatically handles salt

4. **Argon2 (Even Better)**
   - ✅ Modern alternative to Bcrypt

---

## 📊 Encoding vs Encryption vs Hashing

| Feature | Hashing | Encoding | Encryption |
|---------|---------|----------|------------|
| **Purpose** | Password storage, integrity | Format change (transport) | Secrecy |
| **Reversible?** | No | Yes | Yes (with key) |
| **Key needed?** | No | No | Yes |
| **Examples** | Bcrypt, Argon2, SHA-256 | Base64, URL encode | AES, RSA |

### Salt Explained
- Random string generated fresh for each user
- Combined with password before hashing
- Ensures same password ≠ same hash for different users
- Bcrypt bakes salt directly into hash output

**⚠️ Important**: Hashing always happens on **server-side**, never on frontend

---

## 1️⃣ Credential-Based Flows

### A. Basic Authentication
**How it works**: Username:password encoded in Base64 in Authorization header

**Format**: `Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=`

**Flow**:
1. Client requests protected resource (no auth header) → 401 Unauthorized
2. Client sends Base64 encoded credentials
3. Server validates against database
4. If valid → 200 + resource; if invalid → 401

**Characteristics**:
- ❌ No sessions maintained
- ❌ Credentials sent with EVERY request
- ❌ Base64 is NOT encryption (just format change)
- ✅ HTTPS provides actual security
- ✅ Simple and quick

**Best for**:
- Internal admin tools
- Server-to-server API calls
- Quick prototyping
- CI/CD pipelines

---

### B. Digest Authentication
**How it works**: Sends MD5 hash of credentials + server-provided random value (nonce)

**Flow**:
1. Client requests resource → Server responds with 401 + **nonce** (random one-time string)
2. Client generates hash (username + password + nonce)
3. Client sends hash in Authorization header
4. Server validates using stored password hash

**Key improvement**: Raw password never travels over internet (not even Base64)

**Characteristics**:
- ✅ More secure than Basic
- ⚠️ Uses MD5 (outdated)
- ⚠️ Legacy systems only

**Best for**: Legacy enterprise systems, old devices, older HTTP APIs (NOT recommended for new systems)

---

### C. API Key Authentication
**How it works**: Secret key assigned to you, sent with each request

**Common examples**: Google Maps API, OpenAI API, GitHub API, Stripe API

**Flow**:
1. Client requests resource (no API key) → 401
2. Client sends API key in request
3. Server looks up API key in database
4. If valid → 200 + resource; if invalid → 401

**Characteristics**:
- ✅ Simple machine-to-machine authentication
- ❌ No built-in expiry (must rotate manually)
- ❌ If exposed, anyone can use it
- ❌ Requires database lookup each time

**API Key vs JWT**:
| API Key | JWT |
|---------|-----|
| Opaque string | Contains embedded user identity |
| Needs database lookup | Self-contained (no DB lookup needed) |
| No identity info | Has identity & permissions |

**Best for**:
- Third-party developer access
- Server-to-server integrations
- Service APIs (Stripe, SendGrid, OpenAI)

---

## 2️⃣ Session/Token-Based Flows

### A. Session-Based Authentication
**How it works**: Server creates session, client gets session ID in cookie

**Flow**:
1. Client sends login (username + password)
2. Server validates credentials
3. Server creates session with **Session ID** (e.g., ABC123)
4. Server sends Session ID as cookie to client
5. Browser automatically sends cookie with subsequent requests
6. Server validates session → returns protected resource

**Session Storage Options**:
| Option | Pros | Cons |
|--------|------|------|
| Database | Persistent | Slow |
| In-memory | Fast | Lost on restart, doesn't scale |
| **Redis** | Fast, shared across servers | Production standard |
| File system | Simple | Not scalable |

**Characteristics**:
- ✅ Stateful (server holds session state)
- ✅ Automatic cookie handling in browsers
- ⚠️ Must share session across servers in distributed systems

**Best for**:
- Server-rendered web applications
- Applications where client is always a browser

---

### B. Token-Based Authentication (JWT)
**How it works**: Client presents token to access protected resources

**JWT (JSON Web Token) Format**:
- **Header**: Algorithm & token type
- **Payload**: User data (non-sensitive)
- **Signature**: Verifies integrity

**Format in header**: `Authorization: Bearer <token>`

**Flow**:
1. Client sends login (username + password)
2. Server validates credentials
3. Server generates JWT with:
   - User ID, email, roles
   - Expiration time, issue time
   - (Never store sensitive data like passwords)
4. Server returns JWT to client
5. Client sends JWT in Authorization header for subsequent requests
6. Server verifies signature
7. If valid → protected resource; if invalid/expired → 401

**Characteristics**:
- ✅ Self-contained (no database lookup needed)
- ✅ Stateless
- ✅ Has expiry built-in
- ❌ Cannot be revoked before expiry (without additional infrastructure)

---

### C. JWT with Refresh Token
**Why needed**: Access tokens have short expiry (e.g., 5-15 minutes). Refresh tokens prevent repeated logins.

**Flow**:
1. Login request → Server returns **Access Token** + **Refresh Token**
2. Client uses Access Token for API calls
3. When Access Token expires:
   - Client sends Refresh Token to `/refresh` endpoint
   - Server validates Refresh Token
   - Server issues new Access Token
4. Process repeats until Refresh Token expires
5. When Refresh Token expires → user must login again

**Characteristics**:
- ✅ Better user experience (no repeated logins)
- ✅ Security (short-lived access tokens limit damage window)
- ✅ Refresh tokens are long-lived but can be revoked

**Best for**:
- Single Page Applications (React, Angular)
- Mobile applications
- Modern web backends

---

## 3️⃣ Federated/Delegated Flows

### A. OAuth 2.0
**⚠️ Important**: OAuth 2.0 is an **authorization framework**, NOT authentication

**Purpose**: Allow applications to access user resources on another service WITHOUT handling user passwords

**Real-world example**: App that organizes your Google Drive files

**Actors**:
1. **Resource Owner** = User (owns the data)
2. **Client Application** = App wanting access
3. **Authorization Server** = Google's auth server
4. **Resource Server** = Google Drive server

**Flow**:
1. User clicks "Login with Google" on client app
2. Client sends to Google:
   - Client ID (unique app identifier)
   - Redirect URL (where to send user after)
   - Scope (what access is needed)
3. User sees Google login page → enters credentials
4. User sees consent screen → approves access
5. Google sends **authorization code** to client
6. Client exchanges code for **access token**
7. Client uses access token to call Google Drive APIs
8. Google Drive validates token → returns data

**Key benefit**: User NEVER shares Google credentials with third-party app

**Common uses**: "Login with Google/Facebook/GitHub" buttons

---

### B. OpenID Connect (OIDC)
**Definition**: OAuth 2.0 + user identity = OIDC

**What OAuth 2.0 misses**: Identity of the user

**OIDC Enhancement**: Adds **ID Token** (signed JWT containing user identity)

**Flow**:
1. User clicks "Login with Google" (scope includes `openid`)
2. Similar OAuth flow but...
3. After exchanging code, client gets **TWO tokens**:
   - **ID Token** (JWT with user identity: name, email, user ID)
   - **Access Token** (for API calls)
4. Client validates ID Token locally
5. Client uses Access Token for resource server

**When to use which**:
- **OAuth 2.0 only**: Need to use third-party APIs on behalf of user
- **OpenID Connect**: Need to login AND identify the user

---

## 👤 Single Sign-On (SSO)

**Definition**: User experience pattern, NOT an authentication flow

**Concept**: Sign in once, access multiple services from same company without logging in again

**Example**: Google ecosystem - login once, access Gmail, Drive, Maps, etc.

**Flow**:
1. User accesses Gmail → not authenticated → redirected to Google auth server
2. User enters credentials → authenticated
3. User accesses Google Drive
4. Google Drive checks session with auth server
5. If session valid → access granted immediately (no login required)

**Best for**: Companies with multiple products wanting seamless user experience

---

## 🎯 Authentication Decision Guide

| Application Type | Recommended Flow | Why |
|-----------------|-------------------|-----|
| **Internal tools/Admin panels** | Basic Authentication | Simple, quick, few trusted users |
| **Third-party developer APIs** | API Keys | Easy integration, common pattern |
| **Traditional server-rendered web apps** | Session-based (Redis) | Cookie automatic, stateful |
| **SPAs / Mobile backends** | JWT + Refresh Tokens | Stateless, good UX |
| **Login with Google/GitHub** | OpenID Connect | Provides user identity |

---

## 📝 Key Takeaways

1. **Never store passwords in plain text** - use Bcrypt or Argon2 with salt
2. **Hashing happens server-side**, never on frontend
3. **Authentication ≠ Authorization** - know the difference
4. **401 vs 403** - know when to use each
5. **JWT is a token format**, not authentication itself
6. **OAuth 2.0 is authorization**, OpenID Connect adds identity
7. **SSO is user experience**, not a protocol
8. **Always use HTTPS** for security in transit
9. **Match auth flow to your application type** for best results
