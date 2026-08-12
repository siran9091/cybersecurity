# Finding: Broken Access Control – Unauthorized Administrative Account Creation

Severity: Critical
Category: Broken Access Control / Privilege Escalation
OWASP: A01 – Broken Access Control
CWE: CWE-269 – Improper Privilege Management

Description:
The application allows a user to influence the role assigned during the registration process. By modifying the registration request, an administrative role can be supplied even though public registration should create only a standard user account.

The server accepts the modified request and creates the account successfully with administrative privileges. This shows that the backend is trusting a security-sensitive value supplied by the client.

Impact:
An attacker who can access the public registration functionality may create an account with administrator privileges. The attacker can then access administrative functionality and potentially view or modify sensitive application data.

This represents a direct privilege-escalation path from an unprivileged/unauthenticated user to an administrator.

Steps to Reproduce:
1. Open the application's public registration page and enter normal test-user details.
2. Submit the registration request and review the request sent to the server.
3. In the registration JSON, change the role value to the administrative role. The exact test request used in the evidence was:
   "role": "admin"
4. The complete test request used in the screenshot was:

   {
     "email": "test1075@test.com",
     "password": "abc123",
     "passwordRepeat": "abc123",
     "role": "admin",
     "securityQuestion": {
       "id": 2,
       "question": "Mother's maiden name?"
     },
     "securityAnswer": "lisa"
   }

5. Send the modified registration request.
6. The application accepts the request and confirms that the user has been created.
7. Log in with the newly created test account.
8. Open the administration functionality and confirm that the newly created account has administrator-level access.

This demonstrates that the server is accepting a security-sensitive role value supplied by the client instead of assigning the role itself.

Actual Result:
The backend accepts the client-supplied administrative role and creates the account with administrative privileges.

Expected Result:
Public registration must always create a standard user. Role assignment must be controlled by the server and must not be determined by a client-controlled parameter.

Developer Explanation:
The important point is that the issue is not related to an administrator option being visible in the registration page. Even if the UI only shows normal registration, a requester can modify the API request before it reaches the server.

The backend should ignore any client-supplied role or privilege value during normal registration and assign the default user role server-side.

Remediation:
- Do not accept role/privilege assignment from public registration requests.
- Assign the default user role on the server.
- Reject or ignore attempts to submit privileged roles.
- Restrict administrator creation to an authorized provisioning process.
- Enforce authorization checks on every administrative endpoint.
- Add a regression test confirming that public registration cannot create an administrator.

Evidence Guidance:
Place the registration request/response evidence first, followed by evidence showing the newly created account accessing administrator functionality.

Suggested captions:
Figure 1: Modified registration request containing a privileged role.
Figure 2: Application accepts the registration and creates the account.
Figure 3: Newly created account is able to access administrative functionality.

PAYLOAD / REQUEST USED IN THE SCREENSHOT

The registration request shown in the evidence was:

{
  "email": "test1075@test.com",
  "password": "abc123",
  "passwordRepeat": "abc123",
  "role": "admin",
  "securityQuestion": {
    "id": 2,
    "question": "Mother's maiden name?"
  },
  "securityAnswer": "lisa"
}

The important manipulated value is:

"role": "admin"

This is the value that was added/changed to create the account with administrative privileges.

# Finding: Broken Access Control – Unauthorized Access to Administration Section

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
1. Log in with a normal, non-administrative test account.
2. From the application, identify the administration functionality or its underlying route.
3. Directly request the administration route instead of relying on the Administration menu being displayed.
4. The route used during testing was:
   /#/administration
5. Observe the response and the content displayed by the application.
6. Confirm whether the normal user's session is able to access administrative information or functionality.

No special exploit string or payload is required for this finding. The test is simply a direct request using a normal user's authenticated session.

The important point for the application team is that hiding the Administration option from the UI is not sufficient. The server should make the authorization decision for the requested administrative functionality.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

No special exploit payload was used. The administrative functionality was accessed directly through the application route:

/#/administration

For API-level validation, use the authenticated session of the test user and request the corresponding administration endpoint directly. The key point is that the authorization decision must be made by the server.


# Finding: Authentication Bypass Through SQL Injection

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
1. Open the application's login page and use a normal invalid login first to confirm that authentication works as expected.
2. Capture the login request and review the email field.
3. For the authentication-bypass test, replace the email value with the exact value used in the evidence:
   bender@juice-sh.op'--
4. The test request body was:

   {
     "email": "bender@juice-sh.op'--",
     "password": "<test value>"
   }

5. Submit the request.
6. Observe that the application returns a successful authentication response even though the supplied password is not being validated in the normal way.
7. Confirm that an authenticated session is created and that the application identifies the session as the Bender user.

The demonstrated impact is authentication bypass. The finding should therefore be understood as SQL injection leading to authentication bypass, rather than as an authorization bypass.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

The Login Bender test used the following value in the Email field:

bender@juice-sh.op'--

The Password field was populated with a value, but the password is masked in the screenshot and is therefore not reproduced here.

The SQL injection is in the email value above. The trailing `'--` is used to alter the backend authentication query so that the password check is bypassed.

Example request body:

{
  "email": "bender@juice-sh.op'--",
  "password": "<test value>"
}

The report should focus on the demonstrated authentication bypass. Do not describe it as an authorization bypass.

# Finding: Unauthenticated Exposure of Application Metrics

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
1. Start from a session that does not have administrative privileges, or use an unauthenticated browser session.
2. Request the application's metrics endpoint directly:
   GET /metrics
3. No special exploit payload is required; the endpoint itself is the test input.
4. Review the returned response.
5. The response exposes application and process information such as CPU usage, memory usage, startup timing, file-upload statistics, file descriptors and Node.js event-loop metrics.
6. Confirm that the same information is available without the level of access normally expected for internal operational metrics.

The issue is the exposure of the metrics endpoint itself. The finding is not dependent on any SQLi, XSS or other payload.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

No exploit payload was used. The endpoint was requested directly without authentication:

GET /metrics HTTP/1.1
Host: <target>

The response displayed application and process metrics, including CPU, memory, startup timing, file-upload statistics, file descriptors and Node.js event-loop information.


# Finding: Broken Access Control – Feedback Author Spoofing

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
1. Log in as a normal test user and open the customer-feedback functionality.
2. Submit a normal feedback entry and capture the request sent to the server.
3. Review the JSON body and identify the UserId value.
4. During testing, the request was modified to use:
   "UserId": 3
5. The complete test request body shown in the evidence was:

   {
     "UserId": 3,
     "captchaId": 1,
     "captcha": "4",
     "comment": "good (test)@juice-sh.op",
     "rating": 5
   }

6. Send the modified request while remaining logged in as the test user.
7. Observe that the server accepts the request and returns a successful response.
8. Check the resulting feedback entry and confirm that it is associated with the supplied UserId rather than being tied to the authenticated user's identity.

The security issue is that the server trusts the client-supplied UserId when deciding who owns the feedback.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

The request shown in the evidence used the following values:

{
  "UserId": 3,
  "captchaId": 1,
  "captcha": "4",
  "comment": "good (test)@juice-sh.op",
  "rating": 5
}

The important manipulated value is:

"UserId": 3

The authenticated user was able to submit the request while supplying another user's identifier. The server then stored the feedback against that supplied user ID.

Use the exact test email/comment value from the screenshot only if it is required for reproduction; otherwise use a controlled test value.

# Finding: Insecure Direct Object Reference (IDOR) – Unauthorized Access to Another User's Basket

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
1. Log in as a test user and open the shopping basket.
2. Capture the request used to retrieve the basket.
3. Identify the basket identifier in the request.
4. Change the basket identifier to the value demonstrated in the evidence:
   -1
5. The request used during testing was:

   GET /rest/basket/-1 HTTP/1.1
   Host: <target>

6. Send the request using the authenticated test user's existing session.
7. Review the response.
8. Confirm that the response returns basket information belonging to another user instead of restricting access to the authenticated user's own basket.

The important point for the developer is that the server must verify ownership of the requested basket, rather than relying only on the basket ID supplied by the client.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

The basket request used the basket identifier:

GET /rest/basket/-1

The important part of the test was to change the basket ID from the authenticated user's basket to:

-1

The request was made using the authenticated user's existing session. The application then returned another user's basket information.

Example:

GET /rest/basket/-1 HTTP/1.1
Host: <target>
Cookie: <authenticated-user-session>


# 

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
1. Open the application's login page for the test account.
2. During the credential-strength test, the following common password values were checked:

   admin123
   123456
   123456789
   password
   qwerty
   12345678
   12345
   123123
   vin

3. The successful credential shown in the evidence was:
   Username/Email: admin@juice-sh.op
   Password: admin123
4. Submit the login request using the above credential.
5. Observe that authentication succeeds.
6. Confirm the level of access available to the compromised account.

This finding is specifically about the use of a weak/easily guessable password. It should be kept separate from brute-force protection or account-lockout testing.

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

PAYLOAD / REQUEST USED IN THE SCREENSHOT

The screenshot shows a password-strength/credential test using the following candidate values:

admin123
123456
123456789
password
qwerty
12345678
12345
123123
vin

The successful login request shown in the evidence used the account:

admin@juice-sh.op

with the password:

admin123

The report should treat this as a weak-password finding. Do not combine it with the brute-force/lockout finding unless that control was separately demonstrated.

