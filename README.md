# Praxis Middleware: Badger

<div align="center">
  <a href="https://praxis.causq.com/">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/CausQ-Networks/praxis/main/public/logo/praxis-logo-white.png">
      <img alt="Praxis Logo" src="https://raw.githubusercontent.com/CausQ-Networks/praxis/main/public/logo/praxis-logo.png" width="280">
    </picture>
  </a>
</div>

Badger is a middleware plugin designed to work with Traefik in conjunction with [Praxis](https://praxis.causq.com/github), an identity-aware reverse proxy and zero-trust VPN. Badger acts as an authentication bouncer, ensuring only authenticated and authorized requests are allowed through the proxy.

> [!NOTE]
> Badger can also be used standalone for IP handling (Cloudflare and custom proxy support) without Praxis. Simply set `disableForwardAuth: true` in your configuration. See the [Disabling Forward Auth](#disabling-forward-auth) section below for details.

This plugin is **required** to be installed alongside [Praxis](https://praxis.causq.com/github) to enforce secure authentication and session management.

## Installation

Badger is automatically installed with Praxis. Learn how to install Praxis in the [Praxis Documentation](https://docs-praxis.causq.com/self-host/quick-install).

## Configuration

Praxis will provide the necessary configuration to Badger automatically via the HTTP provider. However, you can override the configuration settings by manually providing them in the Traefik config.

### Required Configuration Options

When forward auth is enabled (default), the following options are required:

```yaml
apiBaseUrl: "http://localhost:3001/api/v1"
userSessionCookieName: "p_session_token"
resourceSessionRequestParam: "p_session_request"
```

### Disabling Forward Auth

To disable forward auth and only use IP handling, set `disableForwardAuth: true`. When enabled, all requests pass through without authentication, and the required configuration options above are not needed:

Only do this if you do not need Praxis's authentication features and only want IP handling.

```yaml
disableForwardAuth: true
```

### IP Handling Configuration

Badger automatically extracts the real client IP from requests. By default, it trusts Cloudflare IP ranges and uses the `CF-Connecting-IP` header.

#### Using with Cloudflare (Default)

No additional configuration needed. Badger automatically:

- Trusts Cloudflare IP ranges
- Extracts IP from `CF-Connecting-IP` header
- Sets `X-Real-IP` and `X-Forwarded-For` headers for downstream services

#### Using without Cloudflare

If you're using a different proxy or load balancer, configure custom trusted IPs and/or a custom IP header:

Ensure you always disable the default Cloudflare IP ranges by setting `disableDefaultCFIPs: true` and provide your own trusted IP ranges in CIDR format under `trustip` if using a different proxy.

```yaml
apiBaseUrl: "http://localhost:3001/api/v1"
userSessionCookieName: "p_session_token"
resourceSessionRequestParam: "p_session_request"

# Disable Cloudflare IP ranges
disableDefaultCFIPs: true

# Add your proxy/load balancer IP ranges (CIDR format)
trustip:
  - "10.0.0.0/8"
  - "172.16.0.0/12"

# Optional: Use a custom header instead of CF-Connecting-IP
customIPHeader: "X-Forwarded-For"
```

## Updating Cloudflare IPs

To update the Cloudflare IP ranges, run:

```bash
./updateCFIps.sh
```

This fetches the latest IP ranges from Cloudflare and updates `ips/ips.go`.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
