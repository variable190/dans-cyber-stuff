# Subdomain Takeover

## Summary

Subdomain takeover occurs when a company leaves an orphaned DNS record that points to an external service an attacker can take control of.

## Example

- **Dangling DNS Entries:** A company creates a CNAME or A record pointing to a cloud service, for example:
    -AWS
    - GitHub Pages
    - Azure App Service
- **Abandoned Resources:** The company deletes the cloud resource but does not delete the corresponding DNS record
- **Attacker Claim:** The attacker registers the deleted the cloud resource and serves content under the trusted domain

## Impact

- Phishing from a trusted subdomain
- Session/cookie theft (if cookies are scoped to the parent domain)
- Bypassing CORS or CSP restrictions that trust `*.target.com`
- Hosting malware or fake login pages under a legitimate-looking hostname

## Common services

- AWS (S3, CloudFront, Elastic Beanstalk)
- Azure (App Service, Traffic Manager, Front Door)
- GitHub Pages
- Heroku
- Shopify
- Fastly / other CDNs

## Exploitation

- Enumerate subdomains of the target
- Check each subdomain for dangling DNS (CNAME/A pointing to an external service)
- Confirm the external resource is unclaimed (e.g. GitHub Pages 404, Azure "not found", AWS S3 no such bucket)
- Claim the resource on that service using the same name
- Serve content under the trusted subdomain

## Bug Bounty Report PoC Example

1. Resolve the subdomain:
```bash
dig blog.target.com CNAME +short
```
Result:
texttarget-blog.github.io.

2. Visit `http://blog.target.com`
3. Observe the GitHub Pages default 404 page:
    *"There isn't a GitHub Pages site here."*
4. This confirms the DNS record still points at GitHub Pages, but no repository/user owns `target-blog.github.io.`
5. After claiming the matching GitHub Pages resource, the following proof page is served at `https://blog.target.com`:

```html
HTML<!DOCTYPE html>
<html>
<head>
  <title>Subdomain Takeover PoC</title>
</head>
<body>
  <h1>Subdomain Takeover Proof</h1>
  <p>Host: blog.target.com</p>
  <p>Researcher: [your handle]</p>
  <p>Date: 2026-08-26</p>
  <p>This content is served under a trusted target.com subdomain.</p>
</body>
</html>
```

## Prevention

- Remove old records
- Regular DNS audits
- Prefer provider domain-verification/ownership checks where available

# References

- [mdn - Subdomain takeover](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/Subdomain_takeover)
- [Microsoft - Prevent dangling DNS entries and avoid subdomain takeover](https://learn.microsoft.com/en-us/azure/security/fundamentals/subdomain-takeover)
- [UK Government Security - Potential subdomain takeover](https://www.security.gov.uk/services-resources/cyber-services-government/domain-and-vulnerability-knowledge-base/potential-subdomain-takeover/)

