# Security policy

## Reporting a vulnerability

Please report security issues through a private repository security advisory
addressed to the maintainers. Include the affected version, a minimal
reproduction, and the conditions required to trigger the issue. Do not publish
credentials, private server addresses, or exploit details in a public issue.

The maintainers will acknowledge a report, establish a fix timeline, and
coordinate a release and disclosure date with the reporter.

## Deployment baseline

Run the WebSocket endpoint behind TLS when the browser page is served over
HTTPS. Configure an explicit origin allow-list, set a deployment-specific
access token when authentication is needed, and keep the target host bound to
the local server. Do not expose the plugin as a general TCP proxy.
