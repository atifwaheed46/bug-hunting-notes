# Web Security Fundamentals

## 1. cURL

Reference: [Official cURL manual](https://curl.se/docs/manpage.html)

`curl` is a command-line tool for transferring data to and from servers using URLs. It supports HTTP/HTTPS and other protocols, and is useful for inspecting web responses, sending requests, testing APIs, and downloading files.

```bash
curl --help
curl --help all
curl --help http
```

### Basic requests

```bash
curl https://example.com
curl -I https://example.com
curl -v https://example.com
curl -L https://example.com
curl -k https://example.com
```

- `-I` — request headers only (uses `HEAD`).
- `-v` — verbose details, including request/response headers and connection/TLS information.
- `-L` — follow redirects.
- `-k` / `--insecure` — skip TLS certificate verification. Use only when troubleshooting or testing in a controlled environment; it weakens connection security.

### Headers and authentication

```bash
curl -H "Authorization: Bearer <TOKEN>" https://example.com/api
curl -H "Content-Type: application/json" https://example.com/api
curl -b "session=<COOKIE_VALUE>" https://example.com/account
curl -u '<USERNAME>:<PASSWORD>' https://example.com/
```

- `-H` — add a request header.
- `-b` — send cookies with the request.
- `-u` — use HTTP authentication, commonly Basic authentication.

Avoid putting real passwords or tokens directly in shell history or shared notes.

### Sending data

```bash
# JSON body
curl -X POST https://example.com/api \
  -H "Content-Type: application/json" \
  -d '{"key":"value"}'

# Form-encoded body
curl -X POST https://example.com/login \
  -d 'username=<USER>&password=<PASSWORD>'

# Multipart file upload
curl -F "file=@sample.png" https://example.com/upload
```

- `-d` / `--data` — send request data, usually in the body.
- `-F` / `--form` — send multipart form data, often used for file uploads.
- `-X` — explicitly set the HTTP method. cURL often chooses a suitable method automatically when options such as `-d` are used.

### Download and proxy

```bash
curl -O https://example.com/file.zip
curl -x http://127.0.0.1:8080 https://example.com
```

- `-O` — save the download using the remote filename.
- `-x` / `--proxy` — send traffic through a proxy, for example a local Burp Suite or OWASP ZAP proxy. Configure the proxy listener first.

---

## 2. HTTP basics

Reference: [MDN — Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview) · [MDN — HTTP reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference)

HTTP (Hypertext Transfer Protocol) is an application-layer protocol used for communication between a client (such as a browser or API client) and a server. The client sends a **request**; the server returns a **response**.

A simplified exchange:

```http
GET / HTTP/1.1
Host: example.com
User-Agent: ExampleClient/1.0
Accept: text/html
```

```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 125

<html>...</html>
```

An HTTP request commonly contains a method, path, HTTP version, headers, and sometimes a body. A response commonly contains a version, status code, headers, and sometimes a body.

### URL structure

Reference: [MDN — URI reference](https://developer.mozilla.org/en-US/docs/Web/URI)

Example:

```text
http://user:password@example.com:8080/dashboard.php?login=true#status
```

- `http` — scheme/protocol.
- `user:password@` — optional user information; credentials should not be placed in URLs.
- `example.com` — host/domain.
- `8080` — port.
- `/dashboard.php` — path.
- `?login=true` — query string; starts with `?`, and multiple parameters are commonly separated with `&`.
- `#status` — fragment, normally handled by the browser and not sent as part of the HTTP request to the server.

Query strings often carry search terms, filters, pagination values, or other client-provided parameters. Sensitive information should not be placed in URLs because URLs can be saved in browser history, logs, and referrer data.

### HTTP headers

Reference: [MDN — HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)

Headers carry additional information in requests and responses. Some are mainly used in requests, some in responses, and some concepts apply to both.

**Common request headers**

- `Host` — requested host name in HTTP/1.1.
- `User-Agent` — identifies the client software.
- `Content-Length` — size of the request body in bytes, when applicable.
- `Accept` — response media types the client can handle.
- `Accept-Encoding` — supported compression formats.
- `Cookie` — cookies being sent to the server.
- `Content-Type` — type of data in the request body, such as `application/json`.

**Common response headers**

- `Content-Type` — type of returned content, such as `text/html` or `application/json`.
- `Cache-Control` — caching rules, for example `max-age=3600`.
- `Date` — date and time associated with the response.
- `Set-Cookie` — asks the browser to store a cookie.
- `Location` — commonly identifies the redirect destination.
- `Server` — may identify server software; it can be omitted or customized.

Headers are commonly discussed as request, response, representation/content, general, or security-related headers. These are useful study categories; they are not one universal classification for every HTTP version.

---

## 3. HTTP response status codes

Reference: [MDN — HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)

| Range | Meaning | Example |
|---|---|---|
| `100–199` | Informational | `101 Switching Protocols` |
| `200–299` | Request succeeded | `200 OK`, `201 Created`, `204 No Content` |
| `300–399` | Redirection or cache-related response | `301 Moved Permanently`, `302 Found`, `304 Not Modified` |
| `400–499` | Client/request-side error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed` |
| `500–599` | Server-side error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

Common codes:

- **200 OK** — request succeeded.
- **301 / 302** — redirect; `301` generally means the move is permanent, while `302` is temporary.
- **400 Bad Request** — the server cannot or will not process the request because it considers the request invalid.
- **401 Unauthorized** — authentication is required or has failed.
- **403 Forbidden** — the server refuses the request. Being logged in does not automatically grant permission.
- **404 Not Found** — the requested resource was not found, or the server is choosing not to disclose it.
- **405 Method Not Allowed** — the resource does not support the request method used.
- **500 Internal Server Error** — unexpected server-side failure.

A redirect can send a browser from one URL to another, such as when an HTTP URL redirects to an HTTPS URL.

---

## 4. HTTP methods

Reference: [MDN — HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)

- **GET** — retrieve a representation of a resource. Parameters are often placed in the URL query string, but not every GET request has parameters.
- **POST** — submit data for processing; often used for form submissions, uploads, or creating resources.
- **PUT** — create or replace the state of a resource at a known URI, depending on the API's design.
- **PATCH** — apply a partial update to a resource.
- **DELETE** — request deletion of a resource.
- **HEAD** — like GET, but requests response headers without the response body.
- **OPTIONS** — asks about communication options supported by a resource; often used during CORS preflight.

Data in a POST/PUT/PATCH request is commonly sent in the request body rather than the URL. **POST is not automatically secure**: HTTPS is needed to protect request data in transit. GET query parameters can appear in history and logs, but the method alone does not determine whether data is encrypted.

From a security perspective, sensitive operations such as deleting or changing data must require appropriate authentication and authorization. A method being available does not, by itself, prove a vulnerability.

---

## 5. Cookies and sessions

Reference: [MDN — HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)

HTTP is stateless: each request is independent unless the application uses a mechanism to associate requests with a user or session. Cookies are small pieces of data that a browser stores and may send with later requests.

Typical flow:

1. The browser requests a page.
2. The server responds with a `Set-Cookie` header.
3. The browser stores the cookie, subject to its attributes.
4. On later matching requests, the browser sends it in a `Cookie` header.
5. The server may use the cookie to identify the session or remember a preference.

Example:

```http
Set-Cookie: session=<SESSION_VALUE>; Secure; HttpOnly; SameSite=Lax
```

- `Secure` — send the cookie over HTTPS connections.
- `HttpOnly` — prevents ordinary page JavaScript from reading the cookie.
- `SameSite` — controls when cookies are sent with cross-site requests and can help reduce certain CSRF risks.

A session cookie can be sensitive because possession of it may allow someone to act as the logged-in user. Do not share real session values in notes or screenshots.

---

## 6. HTTPS, TLS, and certificates

Reference: [Cloudflare — What happens in a TLS handshake?](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/) · [MDN — Transport Layer Security](https://developer.mozilla.org/en-US/docs/Web/Security/Transport_Layer_Security)

- **HTTP** carries web requests and responses.
- **HTTPS** is HTTP protected by TLS.
- **TLS (Transport Layer Security)** helps provide confidentiality, integrity, and authentication for data in transit.
- **SSL** is the older predecessor name; modern secure web connections use TLS.
- A **TLS certificate** helps the client verify the server's identity for a domain and supports establishment of the secure connection.

Simplified connection order for HTTP/1.1 or HTTP/2 over TCP:

1. **TCP handshake** — establishes the transport connection.
2. **TLS handshake** — negotiates security settings, validates the certificate, and establishes session keys.
3. **HTTP request/response** — application data is carried inside the encrypted connection.

A valid certificate does not prove that a website itself is trustworthy; it helps verify the identity for the domain and secure the connection. If a certificate is expired, invalid, or for the wrong hostname, clients may warn or refuse the connection.

In cURL, `-k` / `--insecure` disables certificate verification. It is useful only for controlled troubleshooting and should not be used to make normal connections appear safe.

---

## 7. HTTP/2 and HTTP/3 basics

Reference: [Cloudflare — HTTP/3 with QUIC](https://developers.cloudflare.com/speed/optimization/protocol/http3/) · [MDN — Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)

- **HTTP/2** commonly runs over TCP, usually with TLS for HTTPS.
- **HTTP/3** uses **QUIC**, which runs over UDP and integrates TLS 1.3 into its connection setup. HTTPS HTTP/3 commonly uses UDP port `443`.
- HTTP/2 supports multiplexing multiple streams over one connection, but TCP packet loss can temporarily affect delivery of data on that connection.
- QUIC avoids TCP-level head-of-line blocking between independent streams, which can improve performance on lossy networks.
- QUIC uses connection IDs, so a connection can sometimes continue when a device changes networks or its IP address changes.
- QUIC combines connection establishment with TLS setup more tightly; resumed connections may use 0-RTT data, with security considerations.

HTTP/3 does not use TCP. Avoid assuming every website supports HTTP/3; the client and server must negotiate/support it.

---

## 8. Server-Side Template Injection (SSTI)

Reference: [PortSwigger Web Security Academy — Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)

SSTI occurs when untrusted user input is treated as template code instead of being passed safely as data. Template engines include Jinja2, Twig, ERB, and Velocity. The impact depends on the engine and configuration; some SSTI issues may expose data or lead to server-side code execution.

### Basic detection idea

In an authorized lab, submit a harmless mathematical expression to a parameter or input field and check whether the server evaluates it. For example, if the response displays `49` after a suitable expression such as `{{7*7}}`, the template engine may be evaluating user input. Syntax differs by engine, so one payload does not identify every engine conclusively.

Examples shown in the original notes:

```text
Jinja2 / Twig: {{7*7}}
ERB:           <%= 7*7 %>
Velocity:      ${7*7}
```

These are detection examples, not universal syntax for every template engine.

### Jinja2 examples from the original notes

Use command-execution examples only in intentionally vulnerable labs or other explicitly authorized environments:

```text
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('cat flag').read() }}
```

The examples are specific to particular Python/Jinja application contexts and will not work on every Jinja2 target. First establish whether template syntax is being evaluated and identify the engine; do not assume a single expression proves a specific engine.

---

## 9. DNS zone transfers

Reference: [Cloudflare — DNS zone transfers](https://developers.cloudflare.com/dns/zone-setups/zone-transfers/) · [Microsoft Learn — DNS zone transfer](https://learn.microsoft.com/en-us/windows-server/networking/dns/zone-types)

A DNS **zone transfer** synchronizes DNS zone data between authoritative DNS servers.

- **AXFR** — transfers the full DNS zone.
- **IXFR** — transfers changes since an earlier version of the zone.

A zone transfer is normal between authorized DNS servers. It can become an information-disclosure issue if a DNS server permits unauthorized systems to retrieve a zone containing records that should not be exposed.

Useful tools: `dig`, `host`, and `dnsrecon`.

Example for an explicitly authorized domain/server:

```bash
dig AXFR example.com @ns1.example.com
```

Common DNS record types include `A` (IPv4 address), `AAAA` (IPv6 address), `MX` (mail exchanger), `TXT` (text data), `NS` (name server), and `CNAME` (alias). DNS responses and zone-transfer permissions depend on the domain's actual configuration.

---

## 10. File path traversal / directory traversal

Reference: [PortSwigger Web Security Academy — Path traversal](https://portswigger.net/web-security/learning-paths/path-traversal)

Path traversal occurs when an application uses untrusted input to construct a file path without safely restricting which files can be accessed. The sequence `../` means “go up one directory” in many path contexts.

Example path for understanding the idea in a lab:

```text
/var/www/images/../../../../../etc/passwd
```

The exact result depends on the operating system, application, current directory, and path-handling logic.

### Common filtering issues from the original notes

- A filter that removes `../` only once, rather than repeatedly or after canonicalization, may leave a traversal sequence behind. Example to study in a lab: `....//`.
- Applications and proxies may decode URL-encoded input at different stages. Encoding or double encoding only matters when the layers decode it in a way that recreates a traversal sequence; it is not a guaranteed bypass.
- Some applications check that a filename begins in a permitted directory or ends with an expected extension. The validation must be performed on the normalized, canonical path.
- Null-byte extension bypasses such as `%00.png` are historical and runtime-dependent; modern stacks commonly reject or handle null bytes differently.
- Burp Suite Intruder and other fuzzers can help test input variations in authorized labs, but responses must be interpreted rather than assuming every variation is a bypass.

The central concept is whether user-controlled input can make the application read or access a file outside the intended directory. Defensive implementations should normalize paths and enforce a strict allowed base directory.

---

## 11. Nmap basics

Reference: [Official Nmap documentation](https://nmap.org/docs.html)

Nmap is a network discovery and port-scanning tool. Use it only on systems you own or are explicitly authorized to assess.

### Common options

- `-p-` — scan all TCP ports (ports 1–65535).
- `-sS` — TCP SYN scan.
- `-sU` — UDP scan; can be slow, especially across all ports.
- `-sV` — try to identify service/version information.
- `-O` — attempt operating-system detection.
- `-A` — enable several advanced detection features, including OS/version detection, default scripts, and traceroute; can be more intrusive.
- `-T1` through `-T5` — timing templates; higher values are faster but may be less reliable or noisier. `-T4` is not a universal “professional standard”; choose timing based on the environment and authorization.
- `-v` — increase verbosity.
- `--open` — show open (or potentially open) ports in the results.
- `-sC` — run the default set of NSE scripts.
- `-n` — skip DNS resolution.
- `-Pn` — skip host discovery and treat the target as online.
- `--scan-delay <time>` — wait at least the specified time between probes, for example `500ms` or `10s`.

### Examples

```bash
# Discover open TCP ports and identify services on an authorized target
sudo nmap -sS -sV -p- --open -v 192.168.1.1

# More intensive scan; may take longer and generate more traffic
sudo nmap -p- -sS -sU -sV -O -A -T4 -v 192.168.1.1

# Run vulnerability-checking NSE scripts only where explicitly permitted
sudo nmap -p 22,80,443 --script vuln 192.168.1.1

# Disable DNS resolution and host-discovery probes
nmap -n -Pn -sV 192.168.1.1
```

`-sU` combined with `-p-` can make a scan take a long time. NSE scripts vary in behavior, and scripts in the `vuln` category can be intrusive; review what a script does before using it.

Some HTTP NSE scripts accept a `http.useragent` script argument, for example:

```bash
nmap --script-args 'http.useragent=ExampleMobileBrowser' <authorized-target>
```

This changes the User-Agent used by scripts that support that argument; it does not make the entire scan indistinguishable from a mobile browser.

---

## 12. WHOIS and RDAP

Reference: [ICANN — RDAP](https://www.icann.org/rdap/) · [ICANN Lookup](https://lookup.icann.org/)

- **WHOIS** is a query protocol/service historically used to retrieve domain-registration and related registration information. Traditional WHOIS commonly uses TCP port `43`.
- **RDAP (Registration Data Access Protocol)** provides similar registration data through standardized responses, generally in JSON over HTTP(S), and supports discovery of authoritative RDAP services.
- For generic top-level domains, ICANN made RDAP the definitive source for registration data beginning **January 28, 2025**; availability and disclosure of particular fields depend on the registry/registrar and applicable policy.

Registration records may include registrar information, registration dates, name servers, and status values. Privacy services and applicable rules may hide or redact registrant details.

---

## 13. How a web application works

Reference: [MDN — How the web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Web_standards/How_the_web_works)

A web application commonly contains several components:

- **Browser/client:** renders the interface and sends HTTP requests.
- **Frontend:** HTML, CSS, and JavaScript that run in the browser.
- **Backend/application server:** processes requests, applies business rules, checks permissions, and generates responses.
- **Database or other storage:** stores application data.
- **Reverse proxy/load balancer:** may route requests to application servers, terminate TLS, or apply traffic rules.
- **External services:** payment providers, identity providers, object storage, email services, and other APIs.

### Example request flow

When a user opens an order page, a simplified flow might be:

1. The browser requests `GET /orders/517`.
2. The backend identifies the user from a session cookie or access token.
3. The backend checks whether that user is allowed to view order `517`.
4. The backend retrieves permitted data from storage.
5. The server returns HTML or JSON, and the browser renders it.

Example API request:

```http
GET /api/orders/517 HTTP/1.1
Host: shop.example.test
Accept: application/json
Cookie: session=<SESSION_VALUE>
```

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":517,"status":"shipped","total":42.50}
```

**Security idea:** hiding a button in the frontend is not authorization. The backend must check access on every sensitive operation because a user can send requests without using the normal interface.

---

## 14. Request bodies, content types, and encoding

References: [MDN — MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types) · [MDN — POST method and form data](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST) · [MDN — Percent-encoding](https://developer.mozilla.org/en-US/docs/Glossary/Percent-encoding)

The `Content-Type` request header tells the server what format the request body uses. The body format matters when inspecting or modifying web and API requests.

### Common body formats

**JSON — common in APIs**

```http
POST /api/profile HTTP/1.1
Host: app.example.test
Content-Type: application/json

{"displayName":"Atif","timezone":"Asia/Karachi"}
```

Equivalent cURL example for an authorized test endpoint:

```bash
curl -X PATCH 'https://app.example.test/api/profile' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  --data-raw '{"displayName":"Atif"}'
```

**URL-encoded form data — common in HTML forms**

```http
POST /login HTTP/1.1
Host: app.example.test
Content-Type: application/x-www-form-urlencoded

username=atif&remember=true
```

```bash
curl 'https://app.example.test/login' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --data 'username=atif&remember=true'
```

**Multipart form data — commonly used for file uploads**

```bash
curl 'https://app.example.test/api/avatar' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -F 'caption=Profile image' \
  -F 'file=@avatar.png;type=image/png'
```

When using cURL's `-F`, cURL generates the multipart boundary. Usually do not manually set the `Content-Type: multipart/form-data` header, because the boundary parameter must match the generated body.

### Encoding is not encryption

- **Percent/URL encoding** represents characters in URL data, e.g. a space can be represented as `%20` (form encoding commonly uses `+`).
- **Base64** represents bytes using text characters; it does not keep the data secret.
- **Encryption** uses a cryptographic process and keys to protect confidentiality.

Example:

```text
Original value:  hello world
Percent-encoded: hello%20world
Base64:          aGVsbG8gd29ybGQ=
```

Encoding and decoding can happen at multiple layers. During testing, record the exact value sent, how it appears on the wire, and how the application interprets it. Do not assume that double encoding is automatically a bypass; behavior depends on which components decode the input and in what order.

---

## 15. Browser fundamentals and DevTools

References: [MDN — Web APIs](https://developer.mozilla.org/en-US/docs/Web/API) · [Chrome DevTools — Network panel](https://developer.chrome.com/docs/devtools/network/overview) · [Chrome DevTools — Application panel](https://developer.chrome.com/docs/devtools/application)

A browser does more than display HTML. JavaScript can make background requests, change the Document Object Model (DOM), and store data. Security testers should understand where requests originate and where the browser stores information.

### Useful browser concepts

- **DOM:** the in-memory representation of an HTML document that JavaScript can read or modify.
- **`fetch()` / XHR:** browser APIs commonly used by frontend code to send HTTP requests.
- **Cookies:** values that may be attached automatically to matching requests, subject to cookie attributes.
- **`localStorage`:** key/value storage that persists across browser sessions for an origin.
- **`sessionStorage`:** key/value storage scoped to an origin and a browser tab's page session.
- **IndexedDB:** browser-side database storage.
- **Network panel:** shows requests, parameters, cookies, headers, bodies, status codes, timing, and responses.
- **Application/Storage panel:** helps inspect cookies, local storage, session storage, and IndexedDB.

Example JavaScript request:

```javascript
const response = await fetch('/api/profile', {
  headers: { 'Accept': 'application/json' },
  credentials: 'same-origin'
});

console.log('Status:', response.status);
const profile = await response.json();
console.log(profile);
```

In DevTools, open **Network**, reload the page, select the request to `/api/profile`, and inspect its **Headers**, **Payload**, **Preview/Response**, and **Cookies** tabs. This is often the easiest way to learn which API endpoints the frontend actually calls.

**Security idea:** never assume that a value is trusted just because it came from the browser's own frontend. Client-side validation improves user experience but server-side validation and authorization must still be enforced.

---

## 16. Authentication vs. authorization

Reference: [PortSwigger — Access control](https://portswigger.net/web-security/access-control) · [PortSwigger — Authentication](https://portswigger.net/web-security/authentication)

These concepts answer different questions:

- **Authentication:** Who is making the request?
- **Authorization:** Is this authenticated user allowed to perform this action on this resource?

A valid login does not mean the user can access every record or administrative function.

### Example: object-level authorization

Assume the application has two test accounts:

- User A owns order `517`.
- User B owns order `842`.

Both are authenticated. When User A requests their own order:

```http
GET /api/orders/517 HTTP/1.1
Host: shop.example.test
Authorization: Bearer <USER_A_TOKEN>
```

A normal response may be:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":517,"ownerId":1001,"status":"shipped"}
```

For an authorized access-control test, use User A's test token to request User B's test order `842`. The expected secure behavior is denial (often `403 Forbidden` or sometimes `404 Not Found`), without returning User B's data. If the API returns the other user's order without permission, this may indicate broken object-level authorization (often called IDOR/BOLA).

### Example: function-level authorization

A normal user might be allowed to call:

```http
GET /api/profile
```

but not an administrator-only function such as:

```http
GET /api/admin/users
```

The backend must enforce the role/permission requirement on the second endpoint. A hidden admin link in the UI is not a sufficient control.

### Useful distinction

- **Horizontal privilege escalation:** one user accesses another user's data or actions at a similar privilege level.
- **Vertical privilege escalation:** a lower-privileged user accesses a higher-privileged function or role.

Reference: [OWASP API Security Top 10 — 2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)

---

## 17. Sessions, access tokens, JWT, and OAuth

References: [MDN — HTTP authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication) · [PortSwigger — JWT attacks](https://portswigger.net/web-security/jwt) · [OAuth 2.0 Security Best Current Practice (RFC 9700)](https://www.rfc-editor.org/rfc/rfc9700.html)

### Session cookie

A server may keep session state and send the browser a session identifier:

```http
Set-Cookie: session=<RANDOM_SESSION_ID>; Secure; HttpOnly; SameSite=Lax; Path=/
```

The browser may then send:

```http
Cookie: session=<RANDOM_SESSION_ID>
```

The session identifier should be unpredictable, protected in transit, and invalidated appropriately on logout or other security-sensitive events. Treat it like a password: do not publish real values.

### Bearer access token

An API may accept a token in the authorization header:

```http
GET /api/me HTTP/1.1
Host: api.example.test
Authorization: Bearer <ACCESS_TOKEN>
```

A bearer token generally grants access to whoever possesses it, so it must be protected from disclosure. Check token expiry, intended audience, allowed operations/scopes, and authorization decisions on the server.

### JWT basics

A JSON Web Token (JWT) commonly has three Base64URL-encoded parts separated by dots:

```text
header.payload.signature
```

A decoded illustrative payload might look like:

```json
{
  "sub": "user-1001",
  "role": "user",
  "exp": 1900000000,
  "iss": "https://identity.example.test",
  "aud": "example-api"
}
```

The example is illustrative, not a valid credential. Base64URL encoding is not encryption. A signed JWT may be readable by anyone who possesses it; the signature is used to detect unauthorized modification, provided the server validates it correctly. A secure implementation must validate the permitted signing algorithm, signature, expiration, issuer, audience, and any application-specific requirements. The server must not blindly trust a client-modified `role` or other claim.

### OAuth and OpenID Connect

- **OAuth 2.0** is an authorization framework for granting delegated access.
- **OpenID Connect (OIDC)** adds an identity/authentication layer on top of OAuth 2.0.
- An **access token** is for accessing a resource/API.
- An **ID token** in OIDC communicates authentication information to the client.
- A **refresh token** can be used to obtain new access tokens and therefore needs strong protection.

Do not assume that every token is a JWT or that decoding a JWT proves that its signature and claims are valid.

---

## 18. Same-origin policy, CORS, and CSRF

References: [MDN — Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) · [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) · [PortSwigger — CSRF](https://portswigger.net/web-security/csrf)

### Origin and same-origin policy

An origin consists of the **scheme, host, and port**. For example:

- `https://app.example.test` and `https://app.example.test/profile` have the same origin.
- `https://app.example.test` and `https://api.example.test` have different origins because the hosts differ.
- `https://app.example.test` and `http://app.example.test` have different origins because the schemes differ.

The browser's same-origin policy restricts how scripts from one origin can read data from another origin. It does not mean that cross-origin requests can never be sent; the browser may allow some requests while restricting the calling script from reading the response.

### CORS

Cross-Origin Resource Sharing (CORS) is a browser mechanism controlled by response headers. Example response for a frontend hosted at a specifically trusted origin:

```http
Access-Control-Allow-Origin: https://frontend.example.test
Access-Control-Allow-Methods: GET, POST, PATCH
Access-Control-Allow-Headers: Authorization, Content-Type
Vary: Origin
```

For requests that use credentials, the server must configure credential handling explicitly and cannot use `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`. Wildcard or reflected-origin configurations should be evaluated in context; a CORS header alone does not prove a vulnerability. The practical question is whether an untrusted website can read sensitive response data in a victim's browser under realistic conditions.

### CSRF

Cross-Site Request Forgery (CSRF) occurs when a victim's browser is induced to send an unwanted authenticated request to an application. It is most relevant when the application authenticates requests automatically with browser-managed credentials such as cookies.

Conceptual example: a user is logged into `app.example.test`; another website tries to make the browser submit a request that changes the user's email address. If the application relies only on the session cookie and does not appropriately validate the request, it may be vulnerable.

Common defenses include unpredictable CSRF tokens validated by the server, appropriate `SameSite` cookie settings, checking request context where appropriate, and avoiding state changes through safe/read-only methods such as GET. `SameSite` is helpful but should not be treated as the only defense in every application.

**Important:** CORS and CSRF are different. CORS controls browser cross-origin response access; CSRF concerns unwanted requests made with a victim's authority.

---

## 19. REST API fundamentals

References: [PortSwigger — API testing](https://portswigger.net/web-security/api-testing) · [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) · [OWASP API Security Top 10 — 2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)

An API exposes operations that a client can call. In a REST-style API, URLs commonly identify resources, HTTP methods describe the operation, and JSON is often used for request/response bodies.

### Common API components

- **Endpoint:** a URL/path exposed by the API, e.g. `/api/products`.
- **Path parameter:** part of the path, e.g. `/api/products/42`.
- **Query parameter:** a value after `?`, e.g. `/api/products?category=books&page=2`.
- **Request body:** data sent to the API, often JSON.
- **Schema:** expected fields, data types, and constraints.
- **Pagination/filtering/sorting:** mechanisms to return a subset of results.
- **Versioning:** a way to distinguish API versions, e.g. `/api/v1/products`.
- **OpenAPI:** a specification format often used to describe endpoints, parameters, schemas, and responses.

Example request:

```http
GET /api/products?category=books&page=2&limit=20 HTTP/1.1
Host: api.example.test
Accept: application/json
Authorization: Bearer <ACCESS_TOKEN>
```

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "page": 2,
  "limit": 20,
  "items": [
    {"id": 42, "name": "Web Security Basics", "price": 19.99}
  ]
}
```

### Example: update a resource

```bash
curl -X PATCH 'https://api.example.test/v1/profile' \
  -H 'Authorization: Bearer <ACCESS_TOKEN>' \
  -H 'Content-Type: application/json' \
  --data-raw '{"displayName":"Atif"}'
```

Use only your own account or an authorized test account when modifying data.

### API security questions to understand

1. Does the endpoint require authentication when it should?
2. Does the server check whether this user may access this specific object?
3. Can the user read or modify properties they should not control, such as `role`, `ownerId`, or `isAdmin`?
4. Are administrative functions restricted to the correct roles?
5. Are input types, lengths, and allowed values validated server-side?
6. Are response fields limited to the data the caller needs?
7. Are pagination, batch operations, and expensive requests subject to reasonable limits?

Changing an object ID or JSON property is only a test input, not proof of a vulnerability. A finding requires evidence that the server violates an authorization or security rule in an explicitly authorized test.

---

## 20. Burp Suite: inspect and compare HTTP requests

Reference: [Burp Suite documentation — Getting started](https://portswigger.net/burp/documentation/desktop/getting-started) · [Burp Suite — HTTP history](https://portswigger.net/burp/documentation/desktop/tools/proxy/http-history) · [Burp Suite — Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)

Burp Suite is an intercepting proxy used to inspect and modify HTTP/S traffic between a browser/client and a server. Use it on your own applications, training labs, or assets explicitly in scope.

### Basic workflow

1. Start Burp and use its browser, or configure your own browser to use Burp's proxy listener.
2. Visit the application and perform a normal action, such as viewing your profile.
3. Open **Proxy → HTTP history** and select the relevant request.
4. Inspect the method, path, query parameters, headers, cookies, body, status code, and response.
5. Send the request to **Repeater** to repeat a controlled test.
6. Change one value at a time and compare responses. Preserve the original request so you can distinguish the baseline from the modified request.

Example baseline request:

```http
GET /api/profile HTTP/1.1
Host: app.example.test
Cookie: session=<TEST_SESSION>
Accept: application/json
```

Example Repeater experiment in a local lab: change a harmless query parameter from `?page=1` to `?page=2` and note whether the returned page or data changes. Then test a documented invalid value and observe status code, error structure, and validation behavior.

For an access-control lab, use two accounts you control and compare access to the same kind of resource. Do not use someone else's real session, alter real users' records, or continue after the impact is established.

**Practical habit:** record the baseline request, the exact change, the response difference, and what that difference demonstrates. A `200 OK` response by itself is not proof of a security issue.

---

## 21. Reconnaissance and attack surface basics

Reference: [OWASP Web Security Testing Guide — Information gathering](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/) · [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/latest/)

**Reconnaissance** is the process of learning what assets and entry points exist before testing them. The **attack surface** is the set of exposed functionality, endpoints, inputs, services, and trust boundaries that might require security testing.

### Useful items to map

- In-scope domains and subdomains.
- Web applications and API hostnames.
- Login, logout, registration, password reset, and account recovery routes.
- API endpoints, HTTP methods, parameters, and request-body fields.
- User roles and features available to each role.
- JavaScript files that reveal frontend routes or API calls.
- Published API documentation, such as OpenAPI/Swagger, when available.
- DNS records, technologies, and externally exposed services that are in scope.

### Basic, low-impact examples

Check response headers on an in-scope host:

```bash
curl -I https://app.example.test
```

View a site's robots file (this is a crawler hint, not an access-control mechanism):

```bash
curl https://app.example.test/robots.txt
```

A site's sitemap may expose public URLs intended for discovery:

```bash
curl https://app.example.test/sitemap.xml
```

Review the browser's Network history while navigating the site to identify actual frontend API calls. Maintain an asset inventory with the host, path, method, purpose, authentication requirement, and notes on expected behavior.

**Scope rule:** only perform active discovery, fuzzing, or vulnerability tests against assets permitted by the program or owner. A publicly reachable hostname is not automatically authorized for testing.

---

## 22. Common web security response headers

Reference: [OWASP Secure Headers Project](https://owasp.org/projects/secure-headers-project) · [MDN — HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)

Security headers can instruct browsers to apply additional protections. Their usefulness depends on correct configuration and the application's behavior.

### Content-Security-Policy (CSP)

CSP restricts which sources a browser may use for scripts and other resources. Example starting policy (adapt to the actual application; this is not a universal drop-in configuration):

```http
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'self'
```

- `default-src 'self'` sets a default source policy.
- `object-src 'none'` blocks plugin/object content.
- `base-uri 'self'` restricts which origins can be used in a document's `<base>` element.
- `frame-ancestors 'self'` restricts which origins may embed the page in a frame.

A strong CSP can reduce some exploit impact, but it is not a substitute for fixing the underlying injection flaw.

### Strict-Transport-Security (HSTS)

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

HSTS tells a browser to use HTTPS for future requests to a host for the configured period. Enable options such as `includeSubDomains` only after confirming all relevant subdomains support HTTPS.

### X-Content-Type-Options

```http
X-Content-Type-Options: nosniff
```

This asks browsers not to guess a different MIME type in contexts where MIME sniffing could create security problems.

### Frame protection

Modern applications commonly use CSP's `frame-ancestors` directive to control framing. `X-Frame-Options: DENY` or `SAMEORIGIN` is also found in deployed applications and provides older, narrower framing controls.

### Important notes

- `Access-Control-Allow-Origin` is a CORS header, not a general-purpose security header.
- `Server` or `X-Powered-By` disclosures can reveal technology information, but their presence alone is not normally proof of a vulnerability.
- Missing headers should be evaluated against the application, threat model, and policy. A scanner flag is a lead to investigate, not automatically a reportable bug.

---

## 23. DNS resolution and DNS records

Reference: [Cloudflare — What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/) · [MDN — How the web works](https://developer.mozilla.org/en-US/docs/Learn_web_development/Getting_started/Your_first_website/How_the_web_works) · [RFC 1034 — Domain names: concepts and facilities](https://www.rfc-editor.org/rfc/rfc1034) · [RFC 1035 — Domain names: implementation and specification](https://www.rfc-editor.org/rfc/rfc1035)

**DNS (Domain Name System)** translates human-readable names such as `example.com` into information that network clients need to reach services. The most familiar result is an IP address, but DNS also publishes records for mail routing, name servers, aliases, verification, and other services.

### What happens when a browser resolves a domain?

A simplified lookup flow is:

```text
Browser / operating system
          |
          | asks for example.com
          v
Stub resolver
          |
          v
Recursive DNS resolver
          |
          | if the answer is not cached:
          v
       Root server
          |
          v
       .com TLD server
          |
          v
Authoritative DNS server for example.com
          |
          v
DNS answer (for example, an A or AAAA record)
          |
          v
Client connects to the returned IP address
```

The flow is simplified: a device may have a cached answer, a local DNS resolver may be provided by a router or network, and a recursive resolver may already have some records cached. Therefore, every lookup does not necessarily contact the root, TLD, and authoritative servers.

- **Stub resolver:** the small resolver on the client device that sends a query to a configured recursive resolver.
- **Recursive resolver:** obtains an answer on behalf of the client, using its cache and querying DNS servers when needed.
- **Root server:** directs the resolver toward the appropriate top-level-domain (TLD) name servers.
- **TLD server:** provides information about authoritative name servers for a domain under that TLD, such as `.com`.
- **Authoritative DNS server:** publishes the DNS records for a zone and provides authoritative answers for that zone.

DNS resolution and connecting to a web server are separate stages. DNS provides an address or other record; the client then establishes a network connection to the selected service.

### Common DNS record types

| Record | Purpose | Example meaning |
|---|---|---|
| `A` | Maps a name to an IPv4 address | `example.com → 192.0.2.10` |
| `AAAA` | Maps a name to an IPv6 address | `example.com → 2001:db8::10` |
| `CNAME` | Makes a name an alias of another canonical name | `www.example.com → example.com` |
| `MX` | Lists mail servers for a domain, with priorities | Mail for `example.com` is delivered to the listed mail host |
| `NS` | Identifies name servers authoritative for a zone | `ns1.example.net` serves the zone |
| `TXT` | Stores text values used for verification and other policies | SPF, domain verification, or other published text |
| `SOA` | Contains administrative information about a DNS zone | Primary server, responsible mailbox, serial and timers |
| `PTR` | Used for reverse DNS lookup (address to name) | An IP address maps to a hostname |
| `SRV` | Locates a service by protocol, priority, weight, port, and target | A service can advertise the host and port where it runs |

The IP addresses in examples above use documentation ranges; they are not real service addresses.

### Useful DNS commands with `dig`

`dig` sends DNS queries and displays the response, including answer records and useful metadata.

```bash
# Query the default record type (normally A/AAAA, depending on resolver behavior)
dig example.com

# Query specific record types
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com NS
dig example.com TXT
dig example.com SOA

# Show only answer data
dig +short example.com A

# Ask a specific recursive resolver
dig @1.1.1.1 example.com A

# Trace delegation from the DNS root (more queries; useful for learning DNS hierarchy)
dig +trace example.com

# Reverse lookup an IP address
dig -x 192.0.2.10
```

Example answer (illustrative):

```text
example.com.  300  IN  A  192.0.2.10
```

This means the queried name has an A record with a TTL of 300 seconds and an IPv4 address of `192.0.2.10`.

### TTL and caching

**TTL (Time to Live)** tells DNS caches how long a record may be retained before it should be queried again. A TTL of 300 means five minutes. A low TTL can make planned record changes propagate through caches sooner, while a high TTL can reduce repeated DNS queries. Actual refresh behavior also depends on resolver implementation and cache state.

### DNS security and reconnaissance notes

- DNS records can reveal infrastructure relationships, mail providers, verification services, and hostnames.
- A DNS answer is not proof that the referenced host is in scope for a bug bounty program. Confirm the program's asset list and rules before any active testing.
- A successful zone transfer (AXFR) can disclose a zone's records if the server is incorrectly configured. This is the specific issue covered in the earlier **DNS zone transfers** section.
- DNSSEC provides mechanisms for validating DNS data integrity and origin; it does not encrypt DNS queries or hide queried names.
- DNS over HTTPS (DoH) and DNS over TLS (DoT) encrypt the connection between a client/resolver and a DNS resolver. They do not change the meaning of DNS record types.

---

## 24. TCP/IP, ports, sockets, and connections

Reference: [Cloudflare — What is TCP/IP?](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/) · [MDN — Transport layer](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) · [RFC 9293 — Transmission Control Protocol (TCP)](https://www.rfc-editor.org/rfc/rfc9293) · [IANA — Service Name and Transport Protocol Port Number Registry](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)

HTTP is an application-layer protocol. It depends on lower networking layers to move data between a client and a server. Understanding IP addresses, TCP/UDP, ports, and connections makes HTTP traffic easier to troubleshoot and analyze.

### IP address, transport protocol, and port

- **IP address:** identifies a network interface or destination at the network layer. IPv4 addresses look like `192.0.2.10`; IPv6 addresses look like `2001:db8::10`.
- **TCP:** a connection-oriented transport protocol that provides ordered, reliable byte-stream delivery. TCP handles retransmission and sequencing; it does not understand HTTP paths or JSON fields.
- **UDP:** a connectionless datagram transport. It does not provide TCP-style reliable, ordered delivery by itself. Applications can build additional behavior above UDP.
- **Port:** a number used by transport protocols to identify a service or endpoint on a host. TCP port 443 and UDP port 443 are distinct endpoints.
- **Socket:** an operating-system communication endpoint. A TCP connection is commonly identified by the protocol and the source/destination IP addresses and ports (the four-tuple).

A port number does not prove which service is running. For example, HTTPS commonly uses TCP port 443, but a service can run on a nonstandard port, and UDP port 443 may be used by QUIC/HTTP/3.

### TCP three-way handshake

Before normal TCP application data is exchanged, a client and server establish a connection using a three-way handshake:

```text
Client                                      Server
  |                                            |
  | ------------ SYN ------------------------> |
  |                                            |
  | <----------- SYN + ACK ------------------- |
  |                                            |
  | ------------ ACK ------------------------> |
  |                                            |
  |          TCP connection established       |
```

- **SYN:** the client requests to start a TCP connection and provides an initial sequence number.
- **SYN-ACK:** the server acknowledges the client's SYN and provides its own initial sequence number.
- **ACK:** the client acknowledges the server's SYN.

After the handshake, application data can be sent. TCP uses sequence numbers, acknowledgements, checksums, retransmission, and flow/congestion-control mechanisms to provide reliable ordered delivery. A TCP connection is not the same thing as a TLS session or an HTTP request: one connection can carry multiple HTTP requests, depending on the HTTP version and connection behavior.

### Simplified protocol stack

```text
Application layer     HTTP, DNS, SSH, SMTP
          |
Transport layer       TCP or UDP
          |
Internet layer        IP
          |
Link layer            Ethernet, Wi-Fi, etc.
```

This is a simplified model. Real networking stacks and protocol layering can be more detailed.

### Common ports to recognize

| Port | Common service | Important note |
|---:|---|---|
| 22 | SSH | Remote administration; actual service must be verified |
| 25 | SMTP | Mail transfer between systems |
| 53 | DNS | DNS commonly uses both UDP and TCP |
| 80 | HTTP | Common default for unencrypted HTTP |
| 110 | POP3 | Mail retrieval |
| 143 | IMAP | Mail access |
| 443 | HTTPS | HTTPS commonly uses TCP; QUIC/HTTP/3 uses UDP 443 |
| 3306 | MySQL | Common default database port, not guaranteed |
| 5432 | PostgreSQL | Common default database port, not guaranteed |

Ports are conventions, not reliable service identification. Verify a service and version rather than assuming from the port number alone.

### Useful commands

```bash
# Show listening TCP/UDP sockets on Linux
ss -tuln

# Show established TCP connections too
ss -tunap

# Test whether a TCP port is reachable (requires netcat)
nc -vz example.com 443

# Inspect connection details while making an HTTPS request
curl -v https://example.com/
```

Run network scans only against systems you own or have explicit permission to test. A reachable port does not automatically mean a vulnerability exists.

### Relationship with TLS and HTTP versions

- HTTP/1.1 and HTTP/2 are commonly transported over TCP. HTTPS adds TLS for confidentiality, integrity, and server authentication.
- HTTP/3 uses QUIC, which runs over UDP. QUIC integrates transport security using TLS 1.3 mechanisms.
- TCP port 443 and UDP port 443 are not interchangeable. A network path can permit one and block the other.

See the earlier **HTTPS, TLS, and certificates** and **HTTP/2 and HTTP/3 basics** sections for more detail.

---

## 25. HTML forms and JavaScript requests

Reference: [MDN — Sending form data](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data) · [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) · [MDN — HTML forms](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms)

A web application usually collects input in the browser and sends it to a server. HTML forms provide one standard way to submit data. JavaScript can also send requests using APIs such as `fetch()` or the older `XMLHttpRequest` (XHR).

From a security-testing perspective, the important idea is that **the browser is a client, not the authority**. A browser may validate input for usability, but the server must independently validate data and enforce permissions. A user can modify a request before it reaches the server, for example by using browser developer tools, Burp Suite, or cURL.

### Basic HTML form

```html
<form action="/login" method="POST">
  <label>
    Username
    <input type="text" name="username" required>
  </label>

  <label>
    Password
    <input type="password" name="password" required>
  </label>

  <button type="submit">Log in</button>
</form>
```

Important attributes:

- `action` identifies the URL that receives the form submission. If omitted, the browser submits to the current document URL.
- `method` determines the HTTP method, commonly `GET` or `POST`.
- `name` identifies a field in the submitted form data. An input's `id` is mainly for labels, scripting, and CSS; it does not replace `name` in normal form submission.
- `required`, `minlength`, `maxlength`, and `type` can provide client-side validation, but they are not security controls by themselves.

### Form submission formats

The form's `enctype` controls the encoding used for a POST form body:

**1. `application/x-www-form-urlencoded`** (the default for a normal POST form)

```http
POST /login HTTP/1.1
Host: app.example.test
Content-Type: application/x-www-form-urlencoded

username=atif&password=example-value
```

Spaces and special characters are percent-encoded as required by the form encoding rules.

**2. `multipart/form-data`** (commonly used when uploading files)

```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="document">
  <button type="submit">Upload</button>
</form>
```

The browser creates a boundary and separates the body into parts. Each part can have its own headers, such as `Content-Disposition` and a filename. Do not manually set the multipart `Content-Type` boundary when using browser `fetch()` with a `FormData` object; the browser generates it.

**3. `text/plain`**

This is another HTML form encoding option, but it is uncommon for typical application forms and is not a normal substitute for JSON APIs.

### JavaScript `fetch()` request

A frontend can submit JSON without a traditional page reload:

```javascript
async function loadProfile() {
  const response = await fetch('/api/profile', {
    method: 'GET',
    headers: {
      'Accept': 'application/json'
    },
    credentials: 'same-origin'
  });

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }

  const profile = await response.json();
  console.log(profile);
}
```

Example JSON submission:

```javascript
async function updateProfile() {
  const response = await fetch('/api/profile', {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json'
    },
    credentials: 'same-origin',
    body: JSON.stringify({ displayName: 'Atif' })
  });

  const responseBody = await response.json();
  console.log(response.status, responseBody);
}
```

`credentials: 'same-origin'` allows same-origin credentials, such as cookies, to be included. Cross-origin credential behavior is controlled separately and interacts with CORS and cookie settings.

### What to inspect in browser DevTools

Open **Developer Tools → Network**, perform an action in the application, and select the resulting request. Inspect:

- Request URL, method, query string, and status code.
- Request headers, including `Origin`, `Referer`, `Cookie`, `Content-Type`, and `Authorization` when present.
- Request payload or form data.
- Response headers and response body.
- Initiator (which page/script triggered the request).
- Timing, redirects, and whether the request was served from cache.

A visible form field does not necessarily map one-to-one to a backend property. JavaScript may rename fields, add hidden values, combine data, or make several requests. Inspect the actual network request rather than guessing from the page's appearance.

### Security-testing implications

- Client-side validation can be bypassed by editing the request. The server must validate type, length, format, allowed values, and business rules.
- Hidden fields and disabled controls are still client-controlled. The server must not trust a submitted `role`, `price`, `userId`, or similar field without authorization and validation.
- A frontend route or JavaScript file may reveal an API endpoint, but the endpoint must still enforce its own authentication and authorization.
- When comparing behavior, save the baseline request and change one field at a time in an authorized lab or in-scope application.

---

## 26. HTTP caching

Reference: [MDN — HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching) · [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111) · [MDN — Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Cache-Control)

**HTTP caching** stores a copy of a response so it can be reused rather than fetching the same representation from the origin every time. Caches may exist in the browser, an intermediary proxy, a CDN, or another shared infrastructure component.

Caching improves performance and reduces network/server load. It also matters to security because an incorrectly configured shared cache can potentially serve one user's response to another user.

### Freshness and validation

A cached response can generally be reused while it is fresh according to its caching metadata. When it is stale, a cache may contact the origin to validate whether the stored representation is still current.

Two common validation mechanisms are:

- **ETag:** an entity tag identifying a particular representation.
- **Last-Modified:** a timestamp indicating when the representation was last changed.

Example response:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: public, max-age=60
ETag: "profile-v7"

{"version":7,"message":"Public information"}
```

The cache may reuse this response while it is fresh for 60 seconds, subject to the caching rules and request context.

### Conditional request using ETag

After a cached response becomes stale, a client can ask whether its copy is still current:

```http
GET /public-info HTTP/1.1
Host: app.example.test
If-None-Match: "profile-v7"
```

If the representation has not changed, the server can respond:

```http
HTTP/1.1 304 Not Modified
ETag: "profile-v7"
Cache-Control: public, max-age=60
```

`304 Not Modified` tells the cache that it can reuse the stored representation. The 304 response normally does not include the representation body.

If the representation changed, the server normally returns a new response such as `200 OK` with a new body and validator.

### Important Cache-Control directives

| Directive | Meaning |
|---|---|
| `max-age=60` | A response is fresh for 60 seconds, subject to other rules |
| `public` | A shared cache may store the response when other caching rules allow it |
| `private` | The response is intended for a private cache, not a shared cache |
| `no-cache` | A cache may store the response, but must validate it before reuse |
| `no-store` | A cache must not store the request/response under the directive's rules |
| `must-revalidate` | A stale response must be validated before reuse |

**Important distinction:** `no-cache` does not mean "do not store". Use `no-store` when the intention is to prevent storage. Directives can be combined, and request and response directives have related but distinct semantics.

### `Vary` and cache keys

A cache key identifies which request a stored response belongs to. By default, the cache key includes at least the request method and target URI, but caches can use additional request headers where configured. The `Vary` response header tells caches which request headers influenced the selected representation.

Example:

```http
Vary: Accept-Encoding
```

This tells a cache that the response can differ based on the request's `Accept-Encoding` header. A compressed and an uncompressed representation should not be confused.

### Security considerations

- Responses containing account-specific or sensitive data should not accidentally be stored in a shared cache.
- `private` can prevent shared-cache storage, while `no-store` is appropriate when storage itself should be avoided. The right policy depends on the data and application requirements.
- A caching issue should be demonstrated with controlled test accounts and non-sensitive test data. Do not attempt to retrieve another real user's cached content.
- A missing cache header is not automatically a vulnerability. Evaluate the response's sensitivity, the cache architecture, and the actual impact.

### Useful cURL examples

```bash
# Inspect cache-related response headers
curl -sSI https://example.com/public-page

# Send a conditional request (use an ETag observed in a prior response)
curl -i https://example.com/public-page \
  -H 'If-None-Match: "profile-v7"'
```

---

## 27. Reverse proxies, CDNs, and load balancers

Reference: [Cloudflare — What is a reverse proxy?](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/) · [Cloudflare — What is a CDN?](https://www.cloudflare.com/learning/cdn/what-is-a-cdn/) · [NGINX — Reverse proxy](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/) · [MDN — HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)

Modern web applications often have several infrastructure components between the browser and the application code. A browser may connect to a CDN or edge proxy rather than directly to the application server.

### Example architecture

```text
                 Internet client
                       |
                       v
                CDN / Edge network
              (caching, TLS, filtering)
                       |
                       v
                 WAF / Reverse proxy
                       |
                       v
                  Load balancer
                    /      \
                   v        v
             App server 1  App server 2
                    \      /
                     v    v
                    Database
```

This is an illustrative design. A real deployment may combine components, omit some of them, or place them in a different order.

### Forward proxy vs. reverse proxy

- **Forward proxy:** represents clients when they access external servers. The destination server sees the proxy as the connecting client unless other information is provided.
- **Reverse proxy:** represents one or more backend servers to clients. Clients connect to the reverse proxy, which forwards requests to an appropriate backend.

A reverse proxy can provide TLS termination, routing by hostname/path, compression, caching, request limits, and centralized logging. A Web Application Firewall (WAF) may be deployed at this layer, but a WAF is not a replacement for fixing application vulnerabilities.

### CDN (Content Delivery Network)

A CDN distributes content through edge locations closer to users. It can cache eligible static or dynamic responses, terminate TLS, absorb some traffic spikes, and apply traffic/security rules. Whether a particular response is cached depends on the CDN configuration and response/request cache rules; not every request automatically uses a cached response.

### Load balancer

A load balancer distributes requests across multiple backend instances. Common strategies include round-robin, least-connections, and hash-based routing. Health checks can remove an unhealthy backend from service. Some applications also use session affinity ("sticky sessions"), although stateless application design is often easier to scale.

### Forwarded headers and client IP addresses

A proxy may add headers that describe the original request. Common examples include:

```http
Host: app.example.test
X-Forwarded-For: 198.51.100.25
X-Forwarded-Proto: https
X-Forwarded-Host: app.example.test
```

The standardized `Forwarded` header can also carry proxy information:

```http
Forwarded: for=198.51.100.25;proto=https;host=app.example.test
```

These values are only trustworthy when they are set or sanitized by a trusted proxy and the application knows which proxy hops it trusts. A direct client may be able to send a header named `X-Forwarded-For` itself. Therefore, an application should not automatically trust arbitrary client-supplied forwarded headers for IP allowlists, rate limits, audit logs, or security decisions.

### TLS termination

TLS termination means that an edge proxy or load balancer decrypts the client's HTTPS connection. It may then connect to the backend using another encrypted connection or, in some architectures, an internal unencrypted connection. The security of the proxy-to-backend segment must be considered separately from client-to-proxy TLS.

### Simple NGINX reverse proxy example

Illustrative configuration for an application listening on a local backend port:

```nginx
server {
    listen 443 ssl;
    server_name app.example.test;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The TLS certificate/key configuration is omitted. This example requires a valid NGINX configuration and a backend listening on the configured address.

### Why this matters during web security testing

- The public edge may behave differently from the backend application. Record which host and route you tested.
- Host and forwarded headers can influence routing, absolute URL generation, logging, and security controls. Test them only within an authorized scope and understand the expected proxy trust model.
- A CDN or WAF may block or transform some requests. A blocked request does not establish that the backend is secure or vulnerable.
- Do not assume an IP discovered behind a domain is an in-scope target. Follow the program's explicit asset and infrastructure rules.

---

## 28. Input validation and output encoding

Reference: [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html) · [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) · [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html) · [MDN — `textContent`](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)

Web applications receive data from users, browsers, APIs, files, and other systems. **Input validation** checks whether incoming data has an acceptable type, structure, range, length, and meaning. **Output encoding** makes data safe for the context in which it is displayed or embedded.

These are related but different controls:

- Validation asks: "Is this input allowed by the application's rules?"
- Encoding asks: "How should this value be represented so it is treated as data, not interpreted as code, in this output context?"
- Parameterized database queries separate SQL instructions from values rather than concatenating user input into SQL syntax.

### Server-side validation

Client-side validation improves user experience, but a user can modify requests or call an endpoint directly. The server must repeat all security-relevant validation.

For example, suppose an API expects a page number between 1 and 100:

```http
GET /api/products?page=2 HTTP/1.1
Host: app.example.test
Accept: application/json
```

The server should validate that `page` is an integer and within the permitted range. It should define behavior for values such as `page=-1`, `page=abc`, a very large number, a missing value, or repeated parameters. It should not rely on the browser's input field to enforce the rule.

Useful validation principles:

- Prefer allowlists: define what values are permitted instead of trying to enumerate every forbidden value.
- Check data type, length, range, format, and business meaning.
- Validate nested JSON properties as well as top-level fields.
- Apply limits to file size, array length, pagination size, and other resource-consuming inputs.
- Normalize data consistently before validation when the application specification requires it. Be careful not to validate one representation and later interpret a different decoded or normalized representation.
- Enforce authorization separately. A syntactically valid `userId` does not mean the caller is allowed to act on that user.

Example JSON schema-style expectation (illustrative):

```json
{
  "type": "object",
  "required": ["displayName"],
  "properties": {
    "displayName": {
      "type": "string",
      "minLength": 1,
      "maxLength": 50
    }
  },
  "additionalProperties": false
}
```

A schema can help define the accepted shape, but the application must still enforce authorization and business rules.

### Output encoding is context-dependent

The same string can be safe in one output context and unsafe in another. HTML text, HTML attributes, JavaScript strings, CSS values, and URLs have different parsing rules. There is no single encoding function that is correct for every context.

For example, a user submits the literal text:

```text
<em>Hello</em>
```

If the application intends to display this as ordinary text, the browser should show the angle-bracket characters rather than create an emphasized HTML element.

Safer DOM example:

```javascript
const output = document.querySelector('#message');
const userInput = '<em>Hello</em>';

// Treat the value as text, not HTML markup.
output.textContent = userInput;
```

Potentially unsafe when `userInput` is untrusted:

```javascript
// Avoid inserting untrusted strings as HTML.
output.innerHTML = userInput;
```

`innerHTML` parses its value as HTML. It can be appropriate for carefully constructed trusted markup, but inserting untrusted data without context-appropriate protections can create a cross-site scripting (XSS) risk.

### SQL parameters: keep data separate from SQL code

Unsafe pattern (do not construct SQL this way):

```text
query = "SELECT * FROM users WHERE email = '" + userInput + "'"
```

Safer pattern using a parameterized query (generic pseudocode):

```text
query = "SELECT * FROM users WHERE email = ?"
execute(query, [userInput])
```

The database driver sends the value as a parameter rather than treating it as part of the SQL statement. Parameter placeholders and APIs vary by language and database driver. Identifiers such as table or column names often cannot be supplied as ordinary bind parameters; use a strict allowlist when dynamic identifiers are required.

### Encoding is not encryption or validation

- **Encoding** changes representation, such as URL percent-encoding or Base64. It does not make data secret or automatically make it safe.
- **Encryption** protects confidentiality using cryptographic keys.
- **Validation** checks whether a value complies with rules.
- **Escaping/output encoding** prevents data from being interpreted as syntax in a particular context.

For example, Base64-encoding a string does not prevent an application from interpreting it as malicious input after decoding. URL-encoding a value also does not replace server-side validation or context-specific output encoding.

### Practical testing approach

In an authorized lab, identify an input and determine where it is used: database query, HTML text, HTML attribute, URL, command, file path, or another context. Use harmless test strings and observe how the application validates, stores, transforms, and returns the value. A value being reflected in a response is not by itself proof of XSS or injection; its interpretation and impact matter.

---

## 29. HTTP method semantics: safe and idempotent methods

Reference: [RFC 9110 — HTTP Semantics, Methods](https://www.rfc-editor.org/rfc/rfc9110#section-9) · [MDN — HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)

HTTP methods describe the intended semantics of a request. The method name alone does not tell you whether an application is secure, whether a user is authorized, or exactly what the backend implementation will do.

Two important properties are **safe** and **idempotent**.

- **Safe method:** the client does not request a state change. The server may still perform incidental effects such as logging, metrics, or cache updates.
- **Idempotent method:** repeating the same request has the same intended effect on server state as making it once. The response status or body can differ between attempts.

### Common method semantics

| Method | Safe? | Idempotent? | Typical purpose |
|---|---|---|---|
| `GET` | Yes | Yes | Retrieve a representation |
| `HEAD` | Yes | Yes | Retrieve response headers without the response body |
| `OPTIONS` | Yes | Yes | Discover communication options for a resource/server |
| `TRACE` | Yes by HTTP semantics | Yes | Diagnostic loop-back; often disabled |
| `PUT` | No | Yes | Create or replace the target resource state |
| `DELETE` | No | Yes | Remove the target resource association/state |
| `POST` | No | Not generally | Submit data for resource-specific processing |
| `PATCH` | No | Not guaranteed | Apply partial modifications to a resource |

These are protocol semantics. An application can be incorrectly implemented—for example, a `GET` endpoint that deletes data violates the expectation of a safe method.

### Safe does not mean "the server does absolutely nothing"

A GET request may create access logs, update analytics, or increment a metric. It is still considered safe when the client is not asking the server to change application state. The distinction matters because browsers, crawlers, prefetchers, and caches may automatically issue safe requests.

### Idempotency example

Suppose a resource is:

```http
GET /api/profile HTTP/1.1
Host: app.example.test
```

A `PUT` request might replace a profile representation:

```http
PUT /api/profile HTTP/1.1
Host: app.example.test
Content-Type: application/json

{"displayName":"Atif","timezone":"Asia/Karachi"}
```

Sending the same PUT request more than once should leave the resource in the same intended state as sending it once. The server may return `200`, `204`, or another suitable response depending on its implementation.

A POST request might create a new order:

```http
POST /api/orders HTTP/1.1
Host: app.example.test
Content-Type: application/json

{"productId":"P100","quantity":1}
```

Repeating this POST could create two orders unless the application implements duplicate protection or an idempotency-key mechanism. Do not assume that POST is safe to retry.

### HEAD and OPTIONS examples

```bash
# Request headers without the response body
curl -I https://example.com/

# Ask which methods the server advertises for a resource
curl -i -X OPTIONS https://example.com/api/profile
```

A response may include an `Allow` header:

```http
HTTP/1.1 204 No Content
Allow: GET, HEAD, OPTIONS
```

`Allow` describes methods supported by the resource. It is not the same as CORS's `Access-Control-Allow-Methods`, which is part of the browser cross-origin access mechanism.

### Security-testing notes

- Check whether sensitive state-changing operations are incorrectly exposed through GET. This can create risks involving crawlers, prefetching, browser behavior, and cross-site requests.
- Verify authorization for every method. Hiding a button or changing the frontend does not enforce server-side permissions.
- A method being accepted does not automatically indicate a vulnerability. Confirm whether the operation is intended, who can perform it, and what impact it has.
- Use non-destructive test records and authorized accounts when evaluating PUT, PATCH, and DELETE behavior.

---

## 30. WebSockets fundamentals

Reference: [MDN — WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) · [MDN — Writing WebSocket client applications](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications) · [RFC 6455 — The WebSocket Protocol](https://www.rfc-editor.org/rfc/rfc6455)

**WebSocket** is a protocol for two-way communication between a client and server over a connection that remains open. Unlike the usual HTTP request/response pattern, either side can send messages whenever the connection is active. It is useful for chat applications, live notifications, collaboration tools, dashboards, and real-time updates.

### WebSocket URL schemes

- `ws://` — WebSocket without TLS.
- `wss://` — WebSocket protected by TLS.

For production applications, `wss://` is normally expected so traffic is encrypted in transit.

### Traditional HTTP/1.1 upgrade handshake

A common WebSocket connection begins with an HTTP/1.1 Upgrade handshake. The following is an illustrative request:

```http
GET /chat HTTP/1.1
Host: app.example.test
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://app.example.test
```

A successful response commonly looks like:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: <CALCULATED_ACCEPT_VALUE>
```

The `Sec-WebSocket-Accept` value is calculated from the client's key using the protocol's defined process. It is not an authentication token. After the handshake, the connection carries WebSocket frames rather than ordinary HTTP request/response messages.

WebSocket can also be established over HTTP/2 and HTTP/3 using extended CONNECT mechanisms; the HTTP/1.1 `101 Switching Protocols` example is not the only possible establishment method.

### Basic browser JavaScript example

```javascript
const socket = new WebSocket('wss://app.example.test/ws');

socket.addEventListener('open', () => {
  console.log('WebSocket connection opened');
  socket.send(JSON.stringify({ type: 'hello' }));
});

socket.addEventListener('message', (event) => {
  console.log('Message received:', event.data);
});

socket.addEventListener('error', (event) => {
  console.error('WebSocket error:', event);
});

socket.addEventListener('close', (event) => {
  console.log('Connection closed:', event.code, event.reason);
});
```

The browser's `WebSocket` API exposes events such as `open`, `message`, `error`, and `close`. Applications commonly send text (often JSON) or binary messages.

### Important WebSocket concepts

- **Handshake:** the initial negotiation that establishes the WebSocket connection.
- **Frame:** a protocol unit that carries data or control information.
- **Message:** application data delivered to the other side; a message may be carried in one or multiple frames.
- **Ping/Pong:** control frames used to check whether a connection is responsive.
- **Close frame:** used to initiate or acknowledge connection closure.
- **Origin:** identifies the origin of a browser page initiating a connection. Servers should validate expected origins for browser-based applications.
- **Subprotocol:** an application-level protocol selected during the handshake using `Sec-WebSocket-Protocol`.

### WebSocket security fundamentals

A successful handshake does not mean every later message is authorized. The application must authenticate the connection and authorize each sensitive action or resource access represented by a message.

Important controls include:

- Validate the `Origin` header against an explicit allowlist for browser clients. Do not treat Origin as a substitute for authentication; non-browser clients may be able to set headers themselves.
- Authenticate the connection using a secure, appropriate mechanism. Be careful with long-lived sessions and token expiry.
- Authorize individual actions and object access at message level, not only when the socket first connects.
- Validate message type, schema, size, and allowed values. Limit message frequency and resource consumption.
- Use `wss://` in production and avoid logging secrets or full sensitive message contents.
- Close connections when a user logs out, a session is revoked, or authorization is no longer valid, where appropriate.

### Inspecting WebSockets during testing

In browser DevTools, open **Network** and filter for **WS** (the exact filter label can vary by browser). Select the connection to inspect the handshake and messages. Burp Suite also supports WebSocket message history and message editing in supported versions.

For an authorized test, document the connection URL, handshake request/response, authentication context, message format, and the exact action being tested. A message accepted by the server is not automatically a security vulnerability; verify whether it performs an action the user should not be allowed to perform.

---

## 31. HTTP content negotiation

Reference: [MDN — Content negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation) · [MDN — Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Accept) · [MDN — Content-Type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Type) · [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)

**Content negotiation** is how a client and server select a suitable representation of a resource. One URL may be able to return different formats or languages. The client communicates its preferences through request headers, and the server chooses a representation it can provide.

### Important headers

| Header | Direction | Purpose |
|---|---|---|
| `Accept` | Request | Media types the client can process, such as JSON or HTML |
| `Accept-Language` | Request | Preferred natural languages |
| `Accept-Encoding` | Request | Content codings the client can decode, such as gzip or br |
| `Content-Type` | Request or response | Media type of the message body being sent |
| `Content-Encoding` | Usually response; can also describe an encoded request body | Coding applied to the message body, such as gzip |
| `Vary` | Response | Request headers that influenced representation selection and should be considered by caches |

Do not confuse `Accept` with `Content-Type`:

- `Accept: application/json` means the client prefers or can process a JSON response.
- `Content-Type: application/json` means the body in this particular message is JSON.

### Example: requesting JSON

```http
GET /api/products/42 HTTP/1.1
Host: app.example.test
Accept: application/json
```

Possible response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id":42,"name":"Keyboard"}
```

### Example: requesting HTML

```http
GET /products/42 HTTP/1.1
Host: app.example.test
Accept: text/html
```

The same resource path might return an HTML representation if the application supports it:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8

<!doctype html>
<html><body><h1>Keyboard</h1></body></html>
```

A server is not required to support every representation. If it cannot provide an acceptable representation, it may respond with `406 Not Acceptable`, depending on its negotiation behavior.

### Example: sending JSON in a request body

```http
POST /api/products HTTP/1.1
Host: app.example.test
Content-Type: application/json
Accept: application/json

{"name":"Keyboard","quantity":2}
```

Here, `Content-Type` describes the request body format. `Accept` describes the response formats the client can accept.

### Content encoding and compression

A client can advertise supported compression formats:

```http
GET / HTTP/1.1
Host: example.com
Accept-Encoding: gzip, br
```

The server may choose one of the supported codings:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Encoding: br
Vary: Accept-Encoding
```

`Content-Encoding` describes how the response body has been encoded. `Content-Type` still describes the media type of the underlying representation. `Vary: Accept-Encoding` helps caches keep differently encoded variants separate.

### Quality values (`q`)

A client can assign relative preferences using quality values between 0 and 1:

```http
Accept: text/html, application/json;q=0.9, */*;q=0.5
Accept-Language: en-US,en;q=0.9
```

In this example, HTML has the default quality value of 1, JSON has 0.9, and other media types are generally acceptable at 0.5. The server still applies its own available representations and negotiation rules.

### Common status codes related to representation and media type

- `200 OK` — a representation was returned successfully.
- `204 No Content` — the request succeeded and the response has no content body.
- `304 Not Modified` — a conditional request indicates that a cached representation can be reused.
- `406 Not Acceptable` — the server cannot provide a representation acceptable under the request's negotiation preferences.
- `415 Unsupported Media Type` — the server refuses the request because its body format or content encoding is not supported for that operation.

### Useful cURL examples

```bash
# Ask for JSON explicitly
curl -i https://example.com/api/products \
  -H 'Accept: application/json'

# Send JSON and ask for a JSON response
curl -i https://example.com/api/products \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  --data '{"name":"Keyboard"}'

# Advertise a language preference
curl -i https://example.com/ \
  -H 'Accept-Language: en-US,en;q=0.9'
```

### Why content negotiation matters during testing

- An endpoint can behave differently depending on `Accept`, `Content-Type`, or `Accept-Language`.
- A server may expose separate HTML and JSON representations of related data. Check that authorization and data filtering are consistent across representations.
- Incorrect cache configuration around `Vary` can cause one representation to be reused for a request that asked for another representation.
- A `415` response usually points to a request-body media-type or encoding mismatch; a `406` concerns the response representation acceptable to the client.
- A different response format or status code is not itself a vulnerability. Determine whether the difference bypasses a security control or exposes data that should not be available.

