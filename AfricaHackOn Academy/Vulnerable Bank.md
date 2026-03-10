# Vulnerable Bank

Performed a web application penetration test on vulnbank.org, Vulnbank APK [Android Application](https://github.com/Commando-X/vuln-bank-mobile/releases/download/v1.2.1/vulnbankapp.apk) and the Github Repository, [vuln-bank](https://github.com/Commando-X/vuln-bank).

## Findings

### Finding 1: Exposed Endpoints without Authentication

**Severity:** Critical

**Affected Assets:** [Website](https://vulnbank.org/login), Vulnerable Bank Application, APIs
Description: There are several publicly accessible endpoints, including authentication pages, password-recovery workflows, static assets, API routes, and potentially sensitive paths such as administrative interfaces.

**Impact:** Exposed endpoints provide attackers with structural information about the application, making targeted attacks more feasible. Unprotected and sensitive paths may present opportunities for unauthorized access attempts.

**Evidence:**

```sh
dirsearch -u vulnbank.org | tee dirsearch.txt

feroxbuster -u https://vulnbank.org/ -k

nmap -A -p- vulnbank.org
```

**Proof-of-Concept (PoC):**
![Web path scanning with dirsearch](<attachments/Vulnerable Bank/Web path scanning with dirsearch.png>)

![Web path scanning with feroxbuster](<attachments/Vulnerable Bank/Web path scanning with feroxbuster.png>)

![Network mapping to find open ports](<attachments/Vulnerable Bank/Network mapping to find open ports.png>)

![Exposed APIs](<attachments/Vulnerable Bank/Exposed APIs.png>)

![Exposed Console Endpoint](<attachments/Vulnerable Bank/Exposed Console Endpoint.png>)

![Access to Exposed Console Endpoint](<attachments/Vulnerable Bank/Access to Exposed Console Endpoint .png>)

### Finding 2: Hardcoded Credentials

**Severity:** High

**Affected Assets:** Website (https://vulnbank.org/login), Vulnerable Bank Application
Description: The website and application is accessible using default administrative credentials. This could allow unauthorized users to access the admin panel, where they can escalate privileges and disrupt business operations.

**Impact:** An attacker gaining access to the website and application administrative interface allows them to access user information, manage users, create admin accounts and approve loan applications. This compromise can lead to a full web and application compromise, data exfiltration, and operational downtime.

**Evidence:**

```sh
cd /data/data/com.vulnerablebankapp/shared_prefs

cat VulnBankPrefs.xml
```

**Proof-of-concept (PoC):**
![Exposed Default Admin Credentials](<attachments/Vulnerable Bank/Exposed Default Admin Credentials.png>)

![Default Administrative Credentials on Website](<attachments/Vulnerable Bank/Default Administrative Credentials on Website.png>)

![Default Administrative Credentials on Application](<attachments/Vulnerable Bank/Default Administrative Credentials on Application.png>)

![Exposed Database Credentials on GitHub](<attachments/Vulnerable Bank/Exposed Database Credentials on GitHub.png>)

![Database Access with Exposed Credentials in GUI](<attachments/Vulnerable Bank/Database Access with Exposed Credentials in GUI.png>)

![Database Access with Exposed Credentials on CLI](<attachments/Vulnerable Bank/Database Access with Exposed Credentials on CLI.png>)

### Finding 3: SQL injection on login page

**Severity:** High

**Affected Asset:** Website (https://vulnbank.org/login), Vulnerable Bank Application
Description: The application fails to properly validate and sanitize user-supplied input thus causing an attacker to manipulate the backend SQL query by injecting crafted input into a vulnerable parameter.

**Evidence:** Payload `' OR 1=1 –` to bypass the authentication

**Impact:** An attacker could bypass authentication and have unauthorized access to an account thus leading to a potential full database compromise. The ability to alter the logic of a database query.

**Proof of Concept (POC):**
![SQL Injection](<attachments/Vulnerable Bank/SQL Injection.png>)
![Successful SQL Injection](<attachments/Vulnerable Bank/SQL Injection successful.png>)

![Validation of the SQL Injection](<attachments/Vulnerable Bank/Validation of the SQL Injection.png>)

### Finding 4: Transaction Vulnerabilities

#### 4.1 Negative Amount Transfers Possible

**Severity:** Critical

**Affected URL/Parameter:** /api/transfer (POST parameter: amount)

**Description:** The application lacks business logic validation to verify that transaction amounts are positive integers. The system likely checks if the input is a number (passing the type check) and fails to check if the number is greater than zero. By submitting a negative value (e.g., -100), the transactional operation is inverted.

**Impact:**

- Unauthorized Wealth Generation: Subtracting a negative amount results in an addition, effectively crediting the sender's account rather than debiting it.

- Theft/Reversal of Funds: Depending on the implementation, this may debit the recipient (victim) while crediting the attacker, allowing for unauthorized reversals or theft without access to the victim's credentials.

**Proof of Concept (PoC):**
![Negative Amount Request](<attachments/Vulnerable Bank/Negative Amount Request.png>)

![Successful Account Debit](<attachments/Vulnerable Bank/Successful Account Debit.png>)

#### 4.2 No Amount Validation (Input Type Mismatch)

**Severity:** High

**Affected Asset:** /api/transfer (POST parameter: amount)

**Description:** The application fails to properly validate the data type of the amount parameter before processing it. The backend blindly attempts to convert the input into a floating-point number. When a non-numeric string (e.g., "invalid_text") is supplied, the application raises an unhandled Python exception (ValueError: could not convert string to float), resulting in a server-side crash.

**Impact:**

- Application Instability: The unhandled exception triggers an HTTP 500 Internal Server Error, indicating the server process crashed for that specific request.

- Information Disclosure: The error response leaks technical details about the backend logic and technology stack (including the use of Python and Werkzeug), aiding attackers in further reconnaissance.

**Proof of Concept (PoC):**
![Invalid Amount Request](<attachments/Vulnerable Bank/Invalid Amount Request.png>)

![Internal Server Error Response](<attachments/Vulnerable Bank/Internal Server Error Response.png>)

#### 4.3 No Transaction Limits

**Severity:** High

**Affected Asset:** /api/transfer (POST parameter: amount)

**Description:** The application processes transfer requests without enforcing limits or validating the amount against the user's available balance. During testing, a transaction of 1,000,000 was successfully processed, despite the account lacking sufficient funds. This indicates a failure in the business logic to enforce overdraft protection or maximum transaction thresholds.

**Impact:**

- Financial Liability: Users can spend money they do not possess (overdraft), creating immediate financial loss for the platform.

- Bypassing Controls: The lack of upper limits allows malicious actors to move massive sums instantly, bypassing standard Anti-Money Laundering triggers.

**Proof of Concept (PoC):**
![Transfer of funds above card balance](<attachments/Vulnerable Bank/Transfer of funds above card balance.png>)

![AfricaHackOn Academy/attachments/Vulnerable Bank/Successful Transfer.png](<attachments/Vulnerable Bank/Successful Transfer.png>)

#### 4.4 Transaction History Information Disclosure

**Severity:** Medium

**Affected Asset:** /api/transactions (GET parameter: account_id)

**Description:** The application fails to enforce an authentication check on the /api/transactions endpoint. An unauthorized user can remove the Authorization and Cookie headers in the request and still be able to successfully retrieve sensitive financial history directly from the API.

**Impact:**

- Privacy Breach: The exposure of transaction dates, amounts, recipients, and current account balances constitutes a significant privacy breach.

- Information Compromise: Attackers can gather comprehensive financial data for internal users, aiding in sophisticated social engineering attacks or identifying high-value targets.

**Proof of Concept (PoC):**
![Removed Cookie and Authentication Headers](<attachments/Vulnerable Bank/Removed Cookie and Authentication Headers.png>)

![Successful Transactions without Cookies](<attachments/Vulnerable Bank/Successful Transactions without Cookies.png>)

### Finding 5: Session Management

**Severity:** High

**Affected Asset:** [Website](https://vulnbank.org/login), Vulnerable Bank Application
Description: The application uses JSON Web Tokens (JWTs) for session management, but it does not properly validate or protect them. Modifying certain fields inside the JWT did not cause the server to invalidate or reject the token. This indicates that the backend either does not verify the token signature correctly or relies on insecure logic when processing user sessions. In addition, active sessions do not expire when expected. Even after logout actions or long periods of inactivity, previously issued JWTs remained valid and continued to grant access.

**Impact:**

- Unauthorized access: Attackers may gain access to accounts or resources they shouldn’t have.

- Session hijacking: Tokens that never expire allow persistent access even after users log out.

- Long-term compromise: Stolen or leaked tokens remain usable indefinitely due to missing session invalidation.

**Proof of Concept (PoC):**
![Cookie Token before Manipulation](<attachments/Vulnerable Bank/Cookie Token before Manipulation.png>)

![Successful Access after Cookie Manipulation](<attachments/Vulnerable Bank/Successful Access after Cookie Manipulation.png>)

![Account Logout](<attachments/Vulnerable Bank/Account Logout.png>)

![Successful Transaction after log out](<attachments/Vulnerable Bank/Successful Transaction after log out.png>)

### Finding 6: Client and Server-Side Flaws

**Severity:** High

**Affected Asset:** https://vulnbank.org/api/v1/forgot-password and /api/v2/forgot- password.
Evidence: Modifying the API version in the password reset request resulted in an unexpected response containing a 3-digit reset pin. This indicates that the backend accepts downgraded or alternate versions of the endpoint without verifying whether the request should be allowed.

**Impact:**

- Unauthorized access to password reset pins.

- Possible full account takeover.

**Proof of concept (POC):**
![/api/v2/forgot-password](<attachments/Vulnerable Bank/forgot-password API.png>)

![/api/v2/forgot-password](<attachments/Vulnerable Bank/Forgot password api.png>)

### Finding 7: Virtual Card Vulnerabilities

#### 7.1 Plaintext Storage of Virtual Card Details

**Severity:** Critical

**Affected Asset:** Database (e.g., virtual_cards table)
Description: Sensitive customer card data, including the Primary Account Number (PAN), CVV, and expiry date, is stored in the database without adequate encryption. This violates fundamental security principles and industry laws and regulations, such as Payment Card Industry Data Security Standard (PCI DSS).

**Impact:**

- Catastrophic Data Breach: In the event of a successful database breach (e.g., via SQL Injection, backup file compromise, or insider threat), the attacker immediately gains full access to all customer card data in a usable format.

- Direct Financial and Reputational Loss: This exposure leads directly to customer financial loss and subjects the organization to maximum regulatory penalties and severe reputational damage.

**Proof of Concept (PoC):**
![Exposure of the card details](<attachments/Vulnerable Bank/Exposure of the card details.png>)

#### 7.2 No Validation on Virtual Card Limits

**Severity:** High

**Affected URL/Parameter:** /api/card/update (PUT/POST body: limit)

**Description:** The application fails to enforce both minimum and maximum boundary checks on the limit parameter. The API accepts both negative values (e.g., -999999) and high positive values (e.g., 9999999999) without validation. This indicates a missing server-side check to ensure the value falls within a predetermined safe range.

**Impact:**

- Unlimited Spending: A negative value is misinterpreted by the backend system as an extremely large or unlimited credit line, bypassing core risk controls and enabling unlimited purchases.

- Massive Fraud: Allowing an absurdly high limit enables an attacker to immediately facilitate large-scale financial fraud.

**Proof of Concept (PoC):**
![No validation on card limits](<attachments/Vulnerable Bank/No validation on card limits.png>)

## Risk

The assessment revealed that Vulnerable Bank is currently operating in a critical-risk state, with multiple high-impact vulnerabilities affecting core authentication, authorization, financial transactions, and customer data security. Several weaknesses, including default administrative credentials, SQL injection on the login page, unrestricted API access, weak JWT validation, and exposed financial data which allows an attacker to gain full system control with minimal effort. These issues aren’t isolated; they reflect systemic gaps in secure development, access control, encryption, and session management across both the web application and backend services.

If exploited, these vulnerabilities enable attackers to perform unauthorized fund transfers, manipulate account balances, compromise user accounts, extract sensitive card details, and maintain persistent long-term access without detection. This creates a severe risk of large-scale financial fraud, customer data breaches, regulatory non-compliance, including PCI DSS violations, and significant reputational harm. Immediate remediation and a coordinated security overhaul are required to bring the platform to an acceptable risk posture.

## Recommendations

1. **Strengthen Passwords & Access Controls**

Replace all default passwords with strong, unique ones and ensure that only authorized administrators can access management or internal system interfaces.

2. **Improve Input Validation & Error Handling**

Make sure the system checks all data provided by users, especially transaction amounts, which should be positive, valid numbers and within reasonable limits. If incorrect data is entered, the system should reject it politely and not crash.

3. **Enhance Protection of Financial Transactions**

Require extra verification, such as MFA, for high-value transfers and ensure the system prevents users from sending more money than they have. Apply consistent rules for API versions and enforce proper authentication for all sensitive operations.

4. **Secure Database Interactions**

Use safe coding practices such as parameterized queries to prevent attacks involving malicious inputs. Regularly review and update the application’s code to catch issues early.

5. **Protect Sensitive Customer Data**

Encrypt all card information using strong standards, avoid storing CVV codes entirely, and use tokenization whenever possible. Ensure sensitive data is never exposed in API responses and follow the principle of giving users access only to what they strictly need.

6. **Limit System Exposure & Abuse**

Implement rate limits to prevent excessive or abusive requests and review security controls after updates or fixes to ensure they work as intended.
