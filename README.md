# Real IP from Cloudflare Proxy/Tunnel

[![Code Coverage](https://codecov.io/gh/pyksid/cloudflarewarp/branch/master/graph/badge.svg?token=QFGZS5QJSG)](https://codecov.io/gh/pyksid/cloudflarewarp)
[![Code Analysis](https://github.com/pyksid/cloudflarewarp/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/pyksid/cloudflarewarp/actions/workflows/codeql-analysis.yml)
[![Codacy Security Scan](https://github.com/pyksid/cloudflarewarp/actions/workflows/codacy-analysis.yml/badge.svg)](https://github.com/pyksid/cloudflarewarp/actions/workflows/codacy-analysis.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/pyksid/cloudflarewarp)](https://goreportcard.com/report/github.com/pyksid/cloudflarewarp)
[![Build and Test Source](https://github.com/pyksid/cloudflarewarp/actions/workflows/buildAndTest.yml/badge.svg)](https://github.com/pyksid/cloudflarewarp/actions/workflows/buildAndTest.yml)
[![Integration Test](https://github.com/pyksid/cloudflarewarp/actions/workflows/prodTest.yml/badge.svg)](https://github.com/pyksid/cloudflarewarp/actions/workflows/prodTest.yml)

If Traefik is behind a Cloudflare Proxy/Tunnel, it won't be able to get the real IP from the external client as well as other information.

This plugin solves this issue by overwriting the X-Real-IP and X-Forwarded-For with an IP from the CF-Connecting-IP header.  
The real IP will be the Cf-Connecting-IP if request is come from cloudflare ( truest ip in configuration file).  
The plugin also writes the CF-Visitor scheme to the X-Forwarded-Proto. (This fixes an infinite redirect issue for wordpress when using CF[443]->PROXY/TUNNEL->Traefik[80]->WP[80])

## Configuration

### Configuration documentation

Supported configurations per body

| Setting             | Allowed values | Required | Description                                         |
| :------------------ | :------------- | :------- | :-------------------------------------------------- |
| trustip             | []string       | No       | IP or IP range to trust                             |
| disabledefaultcfips | bool           | Yes      | Disable the built in list of CloudFlare IPs/Servers |

### Notes re CloudFlare

One thing included in this plugin is we bundle the CloudFlare server IPs with it, so you do not have to define them manually.  
However on the flip-side, if you want to, you can just disable them by setting `disabledefaultcfips` to `true`.

If you do not define `trustip` and `disabledefaultcfips`, it doesn't seem to load the plugin, so just set `disabledefaultcfips` to `false` and you are able to use the default IP list.

### Enable the plugin

```yaml
experimental:
  plugins:
    cloudflarewarp:
      modulename: github.com/pyksid/cloudflarewarp
      version: v1.3.4
```

### Plugin configuration

```yaml
http:
  middlewares:
    cloudflarewarp:
      plugin:
        cloudflarewarp:
          disabledefaultcfips: false
          trustip: # Trust IPS not required if disabledefaultcfips is false - we will allocate Cloud Flare IPs automatically
            - "2400:cb00::/32"

  routers:
    my-router:
      rule: Path(`/whoami`)
      service: service-whoami
      entryPoints:
        - http
      middlewares:
        - cloudflarewarp

  services:
    service-whoami:
      loadBalancer:
        servers:
          - url: http://127.0.0.1:5000
```

# Testing

[https://github.com/pyksid/cloudflarewarp/tree/master/test](https://github.com/pyksid/cloudflarewarp/tree/master/test)

We have written the following tests in this repo:

- golang linting
- yaegi tests (validate configuration matches what Traefik expects)
- General GO code coverage
- Virtual implementation tests (spin up traefik with yml/toml tests to make sure the plugin actually works)
- Live implementation tests (spin up traefik with the plugin definition as it would be for you, and run the same tests again)

These tests allow us to make sure the plugin is always functional with Traefik and Traefik version updates.
