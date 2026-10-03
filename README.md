# HTTP Security Headers Audit Guide

A practical guide to auditing, configuring, and troubleshooting HTTP security headers for Apache, Nginx, LiteSpeed, Cloudflare, PHP, and WordPress websites.

## Overview

HTTP security headers allow web servers to communicate security-related policies to browsers. Proper configuration can help reduce risks associated with clickjacking, content injection, MIME-type confusion, insecure transport, and unintended information disclosure.

This repository provides practical configuration examples, auditing procedures, troubleshooting guidance, and implementation checklists for website administrators and developers.

## Objectives

* Understand commonly used HTTP security headers.
* Audit response headers on websites and web applications.
* Configure headers on Apache, Nginx, and LiteSpeed.
* Understand Cloudflare-related configuration considerations.
* Implement security headers in PHP applications.
* Apply compatible configurations to WordPress websites.
* Troubleshoot conflicting or duplicated headers.
* Validate changes before deploying them to production.

## Repository Structure

```text
http-security-headers-audit-guide/
├── README.md
├── LICENSE
├── docs/
│   ├── security-headers-explained.md
│   ├── content-security-policy.md
│   ├── hsts-configuration.md
│   ├── apache-configuration.md
│   ├── nginx-configuration.md
│   ├── litespeed-configuration.md
│   ├── cloudflare-configuration.md
│   ├── wordpress-implementation.md
│   └── troubleshooting-guide.md
├── examples/
│   ├── apache-headers.conf
│   ├── nginx-headers.conf
│   └── php-security-headers.php
└── checklists/
    └── security-headers-audit.md
```

## Security Headers Explained

### 1. Content-Security-Policy

The `Content-Security-Policy` (CSP) header controls which sources browsers are permitted to load content from.

It can help mitigate certain cross-site scripting (XSS) and content-injection attacks.

Example:

```http
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'self'
```

**Important:** This is a starting example, not a universal production policy. Applications that use external scripts, payment gateways, embedded content, analytics, or inline scripts may require additional directives.

Test policies using report-only mode before enforcing restrictions on an existing website.

### 2. Strict-Transport-Security

The `Strict-Transport-Security` (HSTS) header instructs supported browsers to use HTTPS for future requests to a domain.

Example:

```http
Strict-Transport-Security: max-age=31536000
```

Only enable HSTS after confirming that HTTPS works reliably across the intended domain.

The `includeSubDomains` directive and browser preload mechanisms require additional planning because they can affect subdomains and may be difficult to reverse quickly.

### 3. X-Content-Type-Options

This header helps prevent browsers from interpreting files as a different MIME type than the server declares.

Example:

```http
X-Content-Type-Options: nosniff
```

Ensure that the application serves appropriate `Content-Type` values before enabling this header.

### 4. X-Frame-Options

This header provides browser support for restricting whether a page can be embedded in a frame.

Example:

```http
X-Frame-Options: SAMEORIGIN
```

For modern applications, CSP's `frame-ancestors` directive offers more flexible framing controls.

Example:

```http
Content-Security-Policy: frame-ancestors 'self'
```

Avoid conflicting policies when both mechanisms are configured.

### 5. Referrer-Policy

The `Referrer-Policy` header controls how much referrer information browsers include in outgoing requests.

Example:

```http
Referrer-Policy: strict-origin-when-cross-origin
```

Choose a policy that balances privacy, application functionality, and analytics requirements.

### 6. Permissions-Policy

The `Permissions-Policy` header allows websites to control access to supported browser features.

Example:

```http
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Only disable features that the application does not need. Review browser support and application requirements before deployment.

## Auditing HTTP Security Headers

Before making changes, inspect the headers currently returned by the website.

### Using cURL

```bash
curl -sSI https://example.com/
```

To follow redirects:

```bash
curl -sSIL https://example.com/
```

To display response headers for a particular endpoint:

```bash
curl -sSI https://example.com/login
```

Replace `example.com` with a domain you own or are authorized to test.

### Browser Developer Tools

1. Open the website in your browser.
2. Open Developer Tools.
3. Select the Network tab.
4. Reload the page.
5. Select the main document request.
6. Review the Response Headers section.
7. Repeat for important pages and redirected URLs.

Different endpoints may return different headers depending on application code, caching, proxies, or web server configuration.

## Apache Configuration

Apache configurations may use virtual-host settings, server configuration files, or `.htaccess`, depending on the hosting environment and enabled modules.

Example for an Apache configuration where `mod_headers` is enabled:

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set X-Frame-Options "SAMEORIGIN"
</IfModule>
```

Before deployment:

* Confirm that `mod_headers` is enabled.
* Check whether the hosting provider permits these directives.
* Test the configuration syntax.
* Check for duplicate headers set by the application or proxy.
* Verify the final response after deployment.

Do not enable HSTS until the site's HTTPS configuration has been verified.

## Nginx Configuration

Example configuration for an Nginx server block:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header X-Frame-Options "SAMEORIGIN" always;
```

After making changes, validate the configuration before reloading:

```bash
nginx -t
```

If validation succeeds, reload Nginx using the appropriate service-management method for your system.

Be aware that Nginx `add_header` inheritance rules can affect existing headers when configuration is placed at different levels.

## LiteSpeed Configuration

LiteSpeed environments may use virtual-host configuration, WebAdmin settings, or supported Apache-compatible directives, depending on the installation and hosting permissions.

Example directives where `.htaccess` support and the headers module are available:

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set X-Frame-Options "SAMEORIGIN"
</IfModule>
```

Implementation considerations:

* Verify that the directives are supported by the installed LiteSpeed version.
* Check whether headers are already configured at the server level.
* Avoid maintaining conflicting settings in multiple locations.
* Purge relevant caches when necessary.
* Test the final response using cURL.

## Cloudflare Configuration

Cloudflare may affect HTTP responses through proxying, caching, redirect rules, response-header transformations, and other enabled features.

When auditing a Cloudflare-proxied website:

1. Inspect the public response using cURL.
2. Review applicable Cloudflare response-header rules.
3. Inspect the origin web server configuration.
4. Check for duplicated or conflicting headers.
5. Verify HTTPS redirects and certificate validity.
6. Retest the public response after changes.

Do not assume that a header is absent from the origin simply because it is missing or modified in the public response. Compare the relevant configuration layers where authorized.

## PHP Implementation

PHP applications can set response headers before sending output.

Example:

```php
<?php

if (!headers_sent()) {
    header('X-Content-Type-Options: nosniff');
    header('Referrer-Policy: strict-origin-when-cross-origin');
    header('X-Frame-Options: SAMEORIGIN');
}
```

Implementation considerations:

* Send headers before output is generated.
* Avoid adding headers after the response has started.
* Check whether the web server or reverse proxy already sets the same headers.
* Avoid changing application-wide behavior without testing affected routes.
* Consider application-specific requirements for framing and external resources.

This example should be adapted to the application architecture rather than copied blindly into production.

## WordPress Implementation

WordPress websites may receive headers from the web server, reverse proxy, security plugins, or custom PHP code.

Before adding custom headers:

1. Audit the existing response headers.
2. Check security plugin settings.
3. Inspect web server and proxy configuration.
4. Identify any duplicate header definitions.
5. Test the homepage, login page, administration area, REST API, and relevant ecommerce routes.

### WordPress-Specific Considerations

* Content Security Policy may interfere with scripts and styles from plugins or themes.
* Ecommerce sites may depend on payment gateways, analytics, and external resources.
* Framing restrictions can affect legitimate embedded content.
* HSTS should be evaluated across all affected subdomains.
* Security headers should not be used as a replacement for updates, access control, backups, or vulnerability remediation.

Make changes in a staging environment whenever possible.

## Troubleshooting Guide

### Headers Are Missing

Potential causes:

* Incorrect configuration file or virtual host.
* Unsupported directives or disabled modules.
* Application responses that override expected behavior.
* Redirects handled by another server or proxy.
* Headers applied only to selected response codes.
* Cached responses containing older configuration.

### Duplicate Headers

Potential causes:

* Headers configured in both the web server and PHP.
* Security plugins adding the same header.
* Reverse proxy or CDN transformations.
* Repeated configuration directives.

Review the final response and consolidate configuration where practical.

### Website Features Stop Working After CSP Changes

Potential causes:

* Required external resources are not allowed.
* Inline scripts or styles are blocked.
* A payment gateway or embedded service is missing from the policy.
* A required connection endpoint is excluded.

Use browser console messages and CSP reports to identify blocked resources. Adjust the policy based on verified application requirements rather than disabling CSP entirely.

### HSTS Causes Access Problems

Potential causes:

* HTTPS is unavailable on an affected hostname.
* A certificate has expired or is invalid.
* `includeSubDomains` affects a subdomain that does not support HTTPS.

Review the scope of the policy carefully before deployment. Browser-cached HSTS policies may continue to apply until their configured expiration or other browser-specific conditions are met.

## Security Headers Audit Checklist

* [ ] Confirm that HTTPS is configured correctly.
* [ ] Inspect response headers on the homepage.
* [ ] Inspect login and authentication endpoints.
* [ ] Review Content Security Policy requirements.
* [ ] Verify MIME-type handling.
* [ ] Review clickjacking protection.
* [ ] Select an appropriate Referrer Policy.
* [ ] Review Permissions Policy requirements.
* [ ] Evaluate HSTS scope and deployment readiness.
* [ ] Check for duplicated headers.
* [ ] Test redirects and relevant HTTP response codes.
* [ ] Test important application features.
* [ ] Validate configuration syntax where applicable.
* [ ] Document the changes and rollback procedure.
* [ ] Recheck the public response after deployment.

## Recommended References

* [OWASP HTTP Security Response Headers Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)
* [MDN Web Docs: HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)
* [MDN Web Docs: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)
* [MDN Web Docs: Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Strict-Transport-Security)
* [Apache mod_headers Documentation](https://httpd.apache.org/docs/2.4/mod/mod_headers.html)
* [Nginx Headers Module Documentation](https://nginx.org/en/docs/http/ngx_http_headers_module.html)
* [Cloudflare Documentation](https://developers.cloudflare.com/)

## Intended Audience

* Linux system administrators
* Web hosting providers
* WordPress developers
* PHP developers
* DevOps engineers
* Website security auditors
* Managed hosting support teams

## Contributions

Contributions are welcome, including:

* Additional web server configuration examples.
* Security header troubleshooting guides.
* WordPress compatibility notes.
* Safe auditing scripts.
* Documentation corrections.
* Deployment and rollback checklists.

Please test configuration examples where practical and clearly document version-specific behavior.

## Disclaimer

This repository is intended for educational purposes and authorized defensive security auditing.

Configuration examples are starting points and may require modifications for individual applications and infrastructure environments. Incorrect security header policies can disrupt application functionality or prevent users from accessing a website.

Test changes in a controlled environment, maintain a rollback plan, and verify application behavior before deploying to production.

## License

This project is distributed under the MIT License. See the `LICENSE` file for details.
