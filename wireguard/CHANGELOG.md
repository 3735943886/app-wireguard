# Changelog

## 0.0.3

- Rewrite the README and documentation for ipTIME (Korean)

## 0.0.2

- Mention ipTIME in the app description, so it shows up when searching
- Point the app URL to this fork

## 0.0.1

First release of the ipTIME redirect fork of the WireGuard app.

- Redirect (DNAT) ports coming in through the tunnel with `redirects`,
  by default `443` and `8883` to the same ports on Home Assistant
- Detect the LAN address of Home Assistant for `server.host` and redirects
- Web UI (ingress) to download each peer's configuration file and QR code,
  with ipTIME setup steps
- `peers.client_dns` to set or leave out DNS in client configurations
- `log_client_config` to print client configurations in the log
- Include `PresharedKey` and configured peer private keys in client
  configurations
- Default host port `51830`, so it can run next to the original app
- Build wireguard-go from the GitHub mirror
