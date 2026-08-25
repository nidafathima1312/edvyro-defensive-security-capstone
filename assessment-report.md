# Defensive Security Assessment Report: Student Portal

**Track:** Cyber Security  
**Task:** Defensive Security Assessment Capstone  
**Author:** Nida Fathima  
**Target Environment:** Localhost / Isolated Virtual Machine (`http://127.0.0.1:3000`)  

---

## 1. Scope, Authorization & Rules of Engagement

### Authorization Statement
This assessment was performed with explicit authorization on a self-hosted, fictional Student Internship Portal running in an isolated environment (`localhost`). No external production systems or third-party networks were scanned or tested.

### Scope of Assessment
- **In-Scope Target:** `http://127.0.0.1:3000` (Local Student Portal API & Frontend)
- **Included Testing Domains:** Configuration security, authentication & session management, input validation, dependency vulnerability scanning, and audit logging.
- **Out-of-Scope:** Denial of Service (DoS) flooding, external API endpoints (SendGrid, AWS), and network-layer sniffing.

### Rules of Engagement (RoE)
1. All testing must remain isolated strictly to local loopback addresses (`127.0.0.1`).
2. No real credentials, secrets, or Personal Identifiable Information (PII) may be used or exposed.
3. Exploitation is restricted to proof-of-concept verification only.

---

## 2. Assessment Methodology & Test Matrix

| Testing Category | Test Performed | Initial Status | Finding Severity |
| :--- | :--- | :--- | :--- |
| **Configuration** | Security header inspection (CORS, HSTS, Content Security Policy) | Fail | **MEDIUM** |
| **Authentication** | Brute-force throttling & generic login errors | Pass | **INFORMATIONAL** |
| **Input Validation** | Unsanitized profile inputs (Stored XSS check) | Fail | **HIGH** |
| **Dependencies** | Vulnerability audit on node modules (`npm audit`) | Fail | **HIGH** |
| **Logging** | Verification of security log generation for access control breaches | Fail | **MEDIUM** |

---

## 3. Vulnerability Findings & Evidence

### Finding V-01: Stored Cross-Site Scripting (XSS) via Profile Bio Endpoint
* **Severity:** High
* **Vulnerability Type:** Unsanitized Input / Output Encoding Failure
* **Description:** The `/api/profile/update` endpoint accepts raw HTML/JavaScript input without server-side sanitization. When rendering the student profile, injected scripts execute automatically in the browser context.
* **Reproduction Steps:**
  1. Authenticate as student `testuser`.
  2. Send a POST request to `/api/profile/update` with payload: `<script>alert('XSS')</script>`.
  3. Load the `/profile` page; the script executes unexpectedly.

### Finding V-02: Known Vulnerable Third-Party Dependencies
* **Severity:** High
* **Vulnerability Type:** Vulnerable Components (OWASP A06:2021)
* **Description:** Running `npm audit` revealed outdated dependency versions containing known remote code execution and denial of service CVE vulnerabilities.
* **Evidence:** Audit report flagged high-severity advisories in package trees.

### Finding V-03: Missing Security Response Headers
* **Severity:** Medium
* **Vulnerability Type:** Security Misconfiguration
* **Description:** HTTP responses lacked critical security headers such as `X-Content-Type-Options`, `X-Frame-Options`, and `Content-Security-Policy`.
* **Evidence:** HTTP response headers captured via cURL inspection lacked `nosniff` and `SAMEORIGIN` attributes.
