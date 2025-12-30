# Web Security Interview Questions

## 1. XSS (Cross-Site Scripting)
**Concept:** Attackers inject malicious scripts into web pages viewed by other users.
- **Reflected XSS**: Script comes from the request (URL parameters).
- **Stored XSS**: Script is stored in the database (e.g., comments section).
- **DOM XSS**: Vulnerability in client-side code.

**Prevention:**
- **Sanitization**: Angular does this automatically for data binding.
- **Content Security Policy (CSP)**.
- Avoid `innerHTML` or `bypassSecurityTrustHtml` unless absolutely necessary.

**Example (CSP Header):**
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;
```

## 2. CSRF (Cross-Site Request Forgery)
**Concept:** Attacker tricks a user into executing unwanted actions on a web application where they are currently authenticated. (e.g., changing password without user knowing).

**Prevention:**
- **Anti-CSRF Tokens**: Server sends a token; client must include it in headers for mutating requests.
- **SameSite Cookie Attribute**: `SameSite=Strict` or `Lax`.

**Example (Set-Cookie Header):**
```http
Set-Cookie: session_id=12345; SameSite=Strict; Secure; HttpOnly
```

## 3. CORS (Cross-Origin Resource Sharing)
**Concept:** A browser security feature that restricts cross-origin HTTP requests.
- By default, browsers block requests to a different domain.
- The *Server* must send specific headers (`Access-Control-Allow-Origin`) to allow the browser to accept the response.

**Example (Server Response Header):**
```http
Access-Control-Allow-Origin: https://my-app.com
Access-Control-Allow-Methods: GET, POST, PUT
```

**Note:** CORS is not a security feature for the server; it protects the *user/browser*.

## 4. OAuth vs OIDC vs SSO
**Concept:**
- **OAuth 2.0 (Authorization)**: "I give this app permission to access my photos on Google." It provides an Access Token. It does *not* tell the app *who* the user is.
  
  **Simple Analogy:** If we use Google OAuth, we can login with our Google account. The login/authentication is handled entirely by Google. If the user is a valid OAuth user, Google returns a token and some necessary details to our application - we never handle the password directly.

- **OIDC (OpenID Connect) (Authentication)**: Built on top of OAuth 2.0. "I want to log in with Google." It provides an ID Token (JWT) containing user info.
- **SSO (Single Sign-On)**: A session/user authentication process that permits a user to use one set of login credentials to access multiple applications. OIDC is a common protocol to implement SSO.

  **Simple Analogy:** Login once, access many applications. For example, when you log in to Google once, you can access Gmail, YouTube, Google Drive, etc., without logging in separately to each one. The Identity Provider maintains your session and provides tokens to other connected apps automatically.

**Flow:**
1. App redirects to Identity Provider (IdP).
2. User logs in at IdP.
3. IdP redirects back with code.
4. App exchanges code for Tokens (Access Token + ID Token).
