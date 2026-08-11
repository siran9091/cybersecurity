# cybersecurity
Security best practices and coding skills

[web hosting](webhosting)
  -[hosting is a service that allows users to publish their website files online. It's usually provided by a web hosting company. 
How it works
A web hosting provider stores a website's files on a server 
The server's resources, like memory, CPU, and bandwidth, are shared among multiple websites 
Anyone with internet access can view the website 
Types of web hosting
Shared hosting
A single server hosts multiple websites, sharing resources among them. This is a cost-effective option for small businesses and personal websites. 
Cloud hosting
A virtual server made up of other web servers pools resources from a network. This allows for scalability and flexibility, and can be cost-effective for businesses. 
Dedicated hosting
A single server is dedicated to a single business customer, who has complete control over the server. This gives the customer more authority and control over their website's files. 
Virtual Private Server (VPS) hosting
A physical server is divided into virtual servers, each of which hosts a website. This is a middle ground between shared and dedicated hosting]. 

![High](https://img.shields.io/badge/Severity-HIGH-red) <b><span style="color:red">This is bold red text</span></b>  \bfAdmin Section:
Finding: Broken Access Control – Unauthorized Access to Administration Section

Severity: High
Category: Broken Access Control
OWASP: A01 – Broken Access Control
CWE: CWE-862 – Missing Authorization

Description:
The application's administration section contains functionality intended only for authorized administrators. Testing identified that the administrative functionality can be requested directly without the application consistently enforcing the required authorization at the server side.

This finding focuses specifically on the protection of the administration functionality. It is separate from the registration finding, which concerns creation of an account with an administrative role.

Impact:
A user without administrative privileges may be able to access administrative functionality and view or perform actions on administrative data.

Depending on the functions exposed, unauthorized access may allow an attacker to view registered users, customer feedback, or perform other administrative operations.

Steps to Reproduce:
1. Authenticate using a normal non-administrative account.
2. Do not use the administrative registration technique described in the separate finding.
3. Request the administration endpoint directly.
4. Observe the server response.
5. Verify whether administrative content or administrative data is returned to the normal user.
6. Where permitted, attempt a non-destructive administrative operation to confirm whether authorization is also missing at the API/action level.

Actual Result:
The administration functionality is accessible without the required administrative authorization.

Expected Result:
The server should verify the authenticated user's permissions before returning the administration page or processing any administrative operation.

Developer Explanation:
Hiding an Administration link from a normal user's menu is not an authorization control. A user can directly call the underlying URL or API endpoint.

Authorization must therefore be checked on the server for every administrative page and every administrative API/action. A normal user should receive an HTTP 403 response when attempting to access administrator-only functionality.

Remediation:
- Implement server-side role/permission checks for every administrative endpoint.
- Return HTTP 403 for authenticated users without the required privilege.
- Do not rely on UI visibility or client-side checks.
- Protect both administrative pages and backend APIs.
- Add automated authorization tests for normal-user access to administrative functions.

Evidence Guidance:
Show the normal-user context and the direct request/response that demonstrates access to the administration functionality.

Suggested caption:
Figure 1: Administrative functionality is accessible without the required administrator authorization.

<b><span style="color:red">SQL Inje:</span></b> SQL Inje:
Finding: Authentication Bypass Through SQL Injection

Severity: Critical
Category: SQL Injection / Authentication Bypass
OWASP: A03 – Injection
CWE: CWE-89 – Improper Neutralization of Special Elements used in an SQL Command

Description:
The login functionality is vulnerable to SQL injection in the authentication input. The application processes attacker-controlled input in a way that changes the intended authentication logic, allowing authentication to succeed without providing the legitimate user's password.

During testing, the login request was manipulated and the application returned a successful authentication response for the targeted account.

The security impact demonstrated here is authentication bypass. It should not be described as an authorization bypass unless unauthorized access to a function belonging to another authenticated role is separately demonstrated.

Impact:
An attacker may be able to authenticate as another application user without knowing that user's password.

If the targeted account has elevated privileges, the attacker may gain access to sensitive functionality and data associated with that account.

Steps to Reproduce:
1. Navigate to the application's login page.
2. Capture a normal login request.
3. Submit an invalid password and confirm that normal authentication fails.
4. Modify the authentication input with the SQL injection test value used during the assessment.
5. Submit the modified request.
6. Observe that the application returns a successful authentication response.
7. Verify that an authenticated session is established and that the application identifies the session as the targeted user.

Actual Result:
The application accepts the manipulated authentication input and establishes an authenticated session without the legitimate password.

Expected Result:
Authentication should only succeed when the submitted credentials are valid. User input must never be able to alter the SQL authentication logic.

Developer Explanation:
The key issue is not simply that SQL syntax can be entered into the login field. The vulnerability is confirmed by the resulting authentication bypass.

The application should use parameterized queries/prepared statements for all database operations. User input should never be concatenated into SQL statements.

Remediation:
- Replace dynamically constructed SQL with parameterized queries/prepared statements.
- Use safe ORM/database APIs where applicable.
- Validate input as an additional defense, but do not rely on input filtering as the primary SQLi protection.
- Return generic authentication errors.
- Add regression tests for SQL injection attempts against all authentication parameters.
- Review other database queries that process user-controlled input.

Evidence Guidance:
Show the normal failed authentication followed by the manipulated request and the successful authentication response. Do not rely only on a Burp Scanner advisory.

Suggested captions:
Figure 1: Normal login attempt fails with invalid credentials.
Figure 2: Manipulated authentication request.
Figure 3: Application establishes an authenticated session despite the invalid password.

<b><span style="color:red">This is bold red text</span></b>Exposed Metrics:
Finding: Unauthenticated Exposure of Application Metrics

Severity: Low
Category: Information Disclosure / Security Misconfiguration
OWASP: A05 – Security Misconfiguration
CWE: CWE-200 – Exposure of Sensitive Information to an Unauthorized Actor

Description:
The application's metrics endpoint is publicly accessible and returns internal application and process information.

The response includes operational information such as file-upload statistics, application startup timing, CPU usage, memory usage, heap information, file-descriptor counts, and Node.js event-loop metrics.

These metrics are useful for internal monitoring but should generally not be exposed to unauthenticated external users.

Impact:
An attacker can use the exposed information to improve reconnaissance of the application and its runtime environment.

The information may help an attacker understand application behavior, identify technologies and operational characteristics, and correlate abnormal behavior with application activity.

The evidence demonstrates information disclosure; it does not by itself demonstrate remote code execution or direct system compromise.

Steps to Reproduce:
1. Request the application's metrics endpoint without administrative privileges.
2. Observe that the endpoint returns monitoring information.
3. Review the response for application and process-level metrics.
4. Confirm that the same information is available without appropriate authentication or network restriction.

Actual Result:
Detailed application and process metrics are returned to an unauthorized requester.

Expected Result:
Metrics should be available only to authorized monitoring systems or appropriately privileged administrators.

Developer Explanation:
The endpoint appears to be intended for monitoring rather than normal application users. Publishing it through the public application interface exposes operational details that are not required by external users.

Remediation:
- Restrict the metrics endpoint to trusted monitoring infrastructure.
- Require authentication where appropriate.
- Apply network allowlisting or internal-only access.
- Avoid exposing detailed process metrics through the public interface.
- Review metric labels and values for application-specific sensitive information.

Evidence Guidance:
Place the metrics response screenshot after the description and impact.

Suggested caption:
Figure 1: Publicly accessible metrics endpoint exposes internal application and process information.


<b><span style="color:red">Fogred Feedback:</span></b> 
Finding: Broken Access Control – Feedback Author Spoofing

Severity: Medium
Category: Broken Access Control / Improper Authorization
OWASP: A01 – Broken Access Control
CWE: CWE-639 – Authorization Bypass Through User-Controlled Key

Description:
The feedback functionality accepts a user identifier supplied by the client when creating a feedback record. The identifier can be modified to another user's identifier, and the server accepts the modified request.

As a result, a user can create feedback that is recorded as belonging to another user.

Impact:
An attacker may be able to impersonate another user when creating feedback or other user-generated records.

This can affect the integrity and trustworthiness of application data and may be used to create misleading records under another user's identity.

Steps to Reproduce:
1. Log in as a normal user.
2. Submit a legitimate feedback request.
3. Capture the request.
4. Modify the user identifier in the request to another valid user's identifier.
5. Send the modified request.
6. Observe the successful response.
7. View the resulting feedback and verify that it is associated with the other user's identity.

Actual Result:
The server trusts the user identifier supplied by the client and creates the feedback under the selected user's identity.

Expected Result:
The application should derive the feedback author from the authenticated session/token instead of trusting a client-supplied identity.

Developer Explanation:
The client should not be allowed to decide who performed an action. The server already knows the authenticated user's identity and should use that identity when creating the feedback record.

If an administrative workflow legitimately needs to create feedback on behalf of another user, that should be implemented as a separate privileged function with an explicit authorization check and audit trail.

Remediation:
- Derive the author/user ID from the authenticated user context.
- Remove the userId/author field from normal feedback requests where possible.
- Ignore conflicting client-supplied identity values.
- Apply authorization checks to any administrator-only impersonation/delegation workflow.
- Add automated tests that modify the user identifier and confirm the server rejects/ignores the change.

Evidence Guidance:
Show the modified request and the resulting record demonstrating that the selected user's identity was accepted.

Suggested captions:
Figure 1: Feedback request contains a client-controlled user identifier.
Figure 2: Application creates the feedback under another user's identity.


<b><span style="color:red">This is bold red text</span></b>VIEW BASKET:
Finding: Insecure Direct Object Reference (IDOR) – Unauthorized Access to Another User's Basket

Severity: High
Category: Broken Access Control / IDOR
OWASP: A01 – Broken Access Control
CWE: CWE-639 – Authorization Bypass Through User-Controlled Key

Description:
The basket functionality allows a user-controlled basket identifier to be used to retrieve another user's shopping basket without verifying that the requested basket belongs to the authenticated user.

During testing, the basket identifier was changed to another valid basket identifier and the application returned the corresponding basket contents.

Impact:
An attacker may be able to access another customer's basket and potentially learn information about products, quantities, or other shopping activity.

If related basket operations are also insufficiently authorized, the same weakness may potentially allow modification of another user's basket. Such modification should be reported only if independently demonstrated.

Steps to Reproduce:
1. Authenticate as User A.
2. Access User A's basket and capture the request.
3. Identify the basket identifier used by the request.
4. Replace the identifier with another valid basket identifier belonging to User B.
5. Send the modified request while remaining authenticated as User A.
6. Observe the response.
7. Confirm that the response contains User B's basket information.

Actual Result:
The application returns another user's basket based only on the supplied basket identifier.

Expected Result:
The application should verify that the requested basket belongs to the authenticated user before returning the data.

Developer Explanation:
The identifier itself is not secret and therefore cannot be treated as an authorization mechanism. Even if a basket ID is difficult to guess, the backend must still verify ownership.

The correct authorization decision should be based on the authenticated user and the requested basket's owner.

Remediation:
- Perform an ownership/authorization check before returning basket data.
- Query the basket using both the authenticated user ID and basket ID.
- Do not rely on object identifiers as proof of authorization.
- Apply the same authorization model to basket update/delete/checkout operations.
- Add automated tests for cross-user object access.

Evidence Guidance:
Show User A's authenticated context, the modified basket identifier, and the response containing User B's basket.

Suggested captions:
Figure 1: Authenticated User A requests a basket using a different user's basket identifier.
Figure 2: Server returns another user's basket information without an ownership check.


<b><span style="color:red">This is bold red text</span></b>PAYBACK TIME:
Finding: Business Logic Flaw – Price Manipulation Through Negative Quantity

Severity: High
Category: Business Logic / Insecure Design
OWASP: A04 – Insecure Design
CWE: CWE-840 – Business Logic Errors

Description:
The shopping basket accepts a negative product quantity and uses that value directly in the order calculation.

During testing, a product quantity was changed to a negative value. The application accepted the value and calculated a negative line-item amount, which resulted in a negative overall order total.

Impact:
An attacker may be able to manipulate the financial calculation of an order.

Depending on the downstream payment, refund, loyalty, order, or accounting logic, this could result in financial loss, invalid orders, incorrect balances, or abuse of application rewards.

Steps to Reproduce:
1. Add a product to the shopping basket.
2. Capture the basket update/request.
3. Change the product quantity to a negative integer.
4. Submit the modified request.
5. Observe that the application accepts the negative quantity.
6. Proceed through the order flow.
7. Observe that the negative quantity is included in the price calculation and produces a negative order total.

Actual Result:
The application accepts a negative quantity and calculates a negative product value/total.

Expected Result:
The server should reject negative quantities and allow only valid quantities defined by the business rules.

Developer Explanation:
The frontend controls are not sufficient to protect this functionality. Even if the UI only provides + and – buttons and prevents a negative value, the request can be changed directly.

The backend must validate quantity before using it in any financial calculation.

Remediation:
- Validate quantity server-side.
- Allow only positive integers within a defined maximum.
- Reject negative, zero, fractional, or excessively large quantities where unsupported.
- Recalculate prices using trusted server-side product prices.
- Validate the final order amount before checkout/payment.
- Add automated tests for negative and boundary values.

Evidence Guidance:
Use the basket screenshot showing the negative quantity followed by the order/checkout evidence showing the resulting negative total.

Suggested captions:
Figure 1: Negative product quantity is accepted in the basket.
Figure 2: Negative quantity causes an invalid negative order total.


<b><span style="color:red">This is bold red text</span></b>WEAK PASSWORD:
Finding: Weak Password Policy

Severity: Medium
Category: Identification and Authentication Failures
OWASP: A07 – Identification and Authentication Failures
CWE: CWE-521 – Weak Password Requirements

Description:
The application permits an easily guessable/common password to be used for an account.

Testing demonstrated that multiple invalid/common password attempts were rejected, but a weak password was accepted and allowed successful authentication.

The finding is therefore focused on password strength. It should not be described as a brute-force or missing lockout issue unless a separate test demonstrates that automated repeated authentication attempts are not adequately protected.

Impact:
Weak passwords increase the likelihood of account compromise through password guessing, credential stuffing, or reuse of passwords exposed in previous breaches.

The risk is higher for accounts that have access to sensitive information or privileged functionality.

Steps to Reproduce:
1. Navigate to the application's login page.
2. Attempt authentication using invalid/common passwords.
3. Confirm that invalid credentials are rejected.
4. Authenticate using the identified weak password.
5. Observe that authentication succeeds.
6. Confirm the privileges and information available to the compromised account.

Actual Result:
A weak/easily guessable password is accepted for authentication.

Expected Result:
The application should enforce an appropriate password-strength policy and prevent commonly used or compromised passwords.

Developer Explanation:
The issue is the quality of the accepted password, not simply the number of login attempts.

A strong password policy should prevent users from selecting passwords that are common, predictable, or known to have been compromised.

Remediation:
- Enforce minimum password strength.
- Block commonly used and known-compromised passwords.
- Encourage long passphrases.
- Use MFA for privileged and sensitive accounts.
- Implement rate limiting/account protection separately to defend against automated guessing.

Evidence Guidance:
Show the relevant authentication evidence without exposing real credentials in the final report. Mask passwords in screenshots.

Suggested caption:
Figure 1: Weak password is accepted and results in successful authentication.


<b><span style="color:red">This is bold red text</span></b>REFLECTED XSS:
Finding: Reflected Cross-Site Scripting (XSS)

Severity: High
Category: Injection – Cross-Site Scripting
OWASP: A03 – Injection
CWE: CWE-79 – Improper Neutralization of Input During Web Page Generation

Description:
User-controlled input is reflected by the application into an HTML response without appropriate output encoding. The reflected value is interpreted in a browser-executable context, allowing attacker-controlled JavaScript to execute.

The final evidence should demonstrate both reflection and execution. A string merely appearing in a JSON response should not be treated as XSS.

Impact:
Successful exploitation could allow an attacker to execute JavaScript in another user's browser in the security context of the affected application.

Depending on the application's functionality, this could allow unauthorized actions through the victim's session, modification of displayed content, phishing, or access to data available to the victim.

Steps to Reproduce:
1. Identify the affected input parameter.
2. Submit a harmless XSS proof-of-concept payload through the affected parameter.
3. Observe that the input is reflected in the application's HTML response.
4. Verify that the output is inserted into an executable browser context without appropriate encoding.
5. Confirm execution of the harmless proof-of-concept in the browser.
6. Capture the request, response, and browser execution evidence.

Actual Result:
The application reflects attacker-controlled input into an executable HTML context without sufficient output encoding, resulting in JavaScript execution.

Expected Result:
User-controlled input should be safely encoded according to its output context so that it is rendered as data rather than executable code.

Developer Explanation:
Reflection alone is not enough to call an issue XSS. The important evidence is that the reflected input reaches an executable browser context.

The application should apply context-appropriate output encoding at the point where data is inserted into HTML, attributes, JavaScript, CSS, or URLs.

Remediation:
- Apply context-specific output encoding.
- Use framework-provided safe templating mechanisms.
- Avoid inserting untrusted data through unsafe DOM APIs such as innerHTML.
- Validate input as defense in depth, but do not rely on filtering alone.
- Deploy an appropriate CSP as an additional defense layer, not as the primary XSS fix.
- Add regression tests for the affected parameter.

Evidence Guidance:
The screenshots should show the payload request, the relevant response context, and the browser execution result. Do not use a CSP header screenshot as the primary proof of XSS.

Suggested captions:
Figure 1: XSS test input submitted through the affected parameter.
Figure 2: Attacker-controlled input reflected into the HTML response.
Figure 3: Harmless JavaScript proof-of-concept executes in the browser.

<b><span style="color:red">This is bold red text</span></b>POISION NULL BYTE:
Finding: Improper File Validation / Poison Null Byte

Severity: Medium
Category: Input Validation / File Handling
OWASP: A05 – Security Misconfiguration
CWE: CWE-20 – Improper Input Validation

Description:
The application's file-handling functionality does not correctly validate and normalize filenames containing a null-byte character.

During testing, a crafted filename containing a null byte was processed by the application and caused the application to return content that should not have been exposed through the affected file endpoint.

The observed behavior indicates that validation and file resolution are being performed inconsistently.

Impact:
An attacker may be able to bypass file-extension or filename validation and access unintended files through the affected functionality.

The practical impact depends on the directories and files reachable through the vulnerable endpoint. Sensitive configuration files, package metadata, source files, or other internal files could become exposed if they are located within the reachable path.

Steps to Reproduce:
1. Identify the application's file-serving endpoint.
2. Submit a normal valid filename and confirm the expected behavior.
3. Modify the filename by introducing a null-byte character and the crafted suffix used during testing.
4. Submit the request.
5. Observe that the application processes the crafted filename instead of rejecting it.
6. Confirm that unintended content is returned.

Actual Result:
The application processes the null-byte-containing filename and returns content that should not be accessible through the endpoint.

Expected Result:
Malformed filenames and null-byte characters should be rejected before file resolution. Only explicitly permitted files should be served.

Developer Explanation:
The backend should validate the complete normalized filename before using it to access the filesystem. Validation performed on only part of the filename can be bypassed when different components of the application or underlying libraries interpret the filename differently.

Remediation:
- Reject null bytes and other invalid filename characters.
- Normalize the path before validation.
- Use an allowlist of files/resources that can be served.
- Keep sensitive files outside the public web root.
- Avoid constructing filesystem paths directly from user input.
- Use safe file-handling APIs and ensure consistent validation across all layers.

Evidence Guidance:
Show the crafted request and the unexpected response. The evidence should demonstrate the actual file/content exposure rather than only showing that a null byte was accepted.

Suggested caption:
Figure 1: Crafted filename containing a null byte is processed and results in unintended file/content exposure.


<b><span style="color:red">This is bold red text</span></b>DIRECTROY LISTING:
Finding: Directory Listing Enabled

Severity: Medium
Category: Security Misconfiguration / Information Disclosure
OWASP: A05 – Security Misconfiguration
CWE: CWE-548 – Exposure of Information Through Directory Listing

Description:
The web server/application exposes a directory listing for a publicly accessible directory.

When the directory is requested, the server returns a browsable list of files rather than denying directory access or returning only explicitly published resources.

The directory location is also discoverable through the application's robots.txt response, which makes the exposed location easier to identify.

Impact:
Directory listing allows an attacker to enumerate files that may not have been intended for public access.

Depending on the contents, this can expose backup files, source files, configuration files, uploaded documents, logs, or other internal resources.

Steps to Reproduce:
1. Request the application's robots.txt file.
2. Identify the referenced directory.
3. Request the directory directly.
4. Observe that the server returns a directory listing.
5. Review the listed files and confirm that they are accessible directly.

Actual Result:
The application/server returns a browsable directory listing to an unauthenticated requester.

Expected Result:
Directory browsing should be disabled and only explicitly intended public files should be accessible.

Developer Explanation:
The issue is not simply that a file is publicly available. The server is allowing users to enumerate the contents of a directory.

This increases the likelihood that forgotten or internal files become exposed.

Remediation:
- Disable directory indexing/listing.
- Store private files outside the public web root.
- Use controlled download endpoints for files that require access.
- Review the exposed directory for backups, configuration files, source code, logs, and sensitive documents.
- Remove unnecessary files from production.

Evidence Guidance:
Show the request to the directory and the response containing the directory listing.

Suggested caption:
Figure 1: Public directory listing exposes files stored under the application directory.

<b><span style="color:red">This is bold red text</span></b> STACK TRACE ERROR:
Finding: Stack Trace Information Disclosure

Severity: Medium
Category: Information Disclosure / Improper Error Handling
OWASP: A05 – Security Misconfiguration
CWE: CWE-209 – Generation of Error Message Containing Sensitive Information

Description:
The application returns a detailed server-side stack trace when malformed input is submitted.

During testing, invalid JSON input caused the application to return an HTTP 500 response containing internal implementation details such as source-code paths, module names, framework/runtime information, and code locations.

Impact:
Detailed stack traces provide useful reconnaissance information to an attacker.

Source paths, module names, framework details, and code locations can help an attacker understand the application's internal architecture and identify additional attack opportunities.

Steps to Reproduce:
1. Capture a valid JSON request to the affected API endpoint.
2. Modify the request so that the JSON becomes syntactically invalid.
3. Send the malformed request.
4. Observe the HTTP 500 response.
5. Review the response body and identify the detailed stack trace/internal paths.

Actual Result:
The application exposes a detailed internal stack trace to the client.

Expected Result:
The client should receive a controlled generic error message, while detailed exception information should be written only to protected server-side logs.

Developer Explanation:
The detailed stack trace is useful to developers during troubleshooting, but it should not be returned to an external requester.

The application can preserve the full exception in server logs while returning a generic message such as "Invalid request format" to the client.

Remediation:
- Implement centralized exception handling.
- Return generic client-facing error messages.
- Log detailed exceptions securely on the server.
- Disable debug/verbose error output in production.
- Add negative tests for malformed JSON and other invalid input.

Evidence Guidance:
Place the malformed request before the response screenshot so the developer can clearly understand what triggered the error.

Suggested captions:
Figure 1: Malformed JSON request sent to the affected endpoint.
Figure 2: HTTP 500 response exposes the server-side stack trace.
