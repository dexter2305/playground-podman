# Playground podman

# 02-Haproxy
- Add `127.0.0.1 web.local` to `/etc/hosts`
- `curl web.local:8080/` sends the Host header as "web.local:8080". Hence, the host header match includes the port.
