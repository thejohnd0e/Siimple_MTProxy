# Siimple MTProxy

![Platform](https://img.shields.io/badge/platform-Debian%20%7C%20Ubuntu-0A66C2)
![Shell](https://img.shields.io/badge/script-Bash-2EA44F)
![Docker](https://img.shields.io/badge/deployment-Docker-2496ED)
![Telegram](https://img.shields.io/badge/for-Telegram%20MTProxy-26A5E4)

One-click installation script for deploying an MTProxy server with Docker.

This project is designed to make MTProxy setup as simple as possible: run one command, answer a couple of prompts, and get a ready-to-use Telegram proxy link.

## Quick Start

Run the installer on your server:

```bash
wget -qO- https://raw.githubusercontent.com/thejohnd0e/Siimple_MTProxy/refs/heads/master/install-MTProxy | bash
```

Important:

- Run it as `root`
- Use a Debian-based Linux server with `apt`
- Make sure the selected port is open on your server/provider firewall

## Disclaimer: What MTProxy Is And How To Use It

`MTProxy` is a proxy protocol designed for Telegram. It helps users connect to Telegram through an intermediate server, which can be useful in regions or networks where direct access is limited, filtered, or unstable.

This project does not modify Telegram and does not provide anonymity in the same way as a VPN or Tor. Its purpose is simple: to make it easier to deploy your own Telegram MTProxy server.

Use MTProxy if you want to:

- provide Telegram access through your own server
- share a private proxy link with friends, community members, or users
- keep deployment simple with a one-command installer

Basic usage:

1. Run the installer on a Linux server.
2. Choose a port and a TLS masking domain, or use the defaults.
3. Wait for the script to finish and copy the generated `tg://proxy...` link.
4. Open that link on a device with Telegram installed.
5. Confirm the connection inside Telegram.

Please use this project responsibly and in accordance with the laws, regulations, and platform rules that apply in your country or hosting environment. You are responsible for how your server is deployed and used.

## Important Note About Telegram Voice Calls

Telegram architecture does NOT allow calls via MTProxy, but only via SOCKS5, which cannot be obfuscated.

In practice, this means:

- `MTProxy` can help users connect to Telegram messaging services
- `MTProxy` should not be expected to make Telegram voice calls work
- if voice calls are required, users typically need a different proxy approach such as `SOCKS5` or use VPN.

## What The Script Does

The installer will:

- Check for `root` privileges
- Install Docker if it is missing
- Install the Docker Compose plugin if it is missing
- Ask you for:
  - proxy port
  - TLS masking domain
- Generate a random MTProxy secret
- Create the configuration in `/opt/telemt`
- Start the proxy in Docker
- Try to extract and display the `tg://proxy...` connection link from the logs

## Default Values

If you just press Enter during setup, the script uses:

- Port: `8443`
- TLS domain: `www.google.com`

## Files Created

The installer creates the following structure:

```text
/opt/telemt/
|-- docker-compose.yml
`-- telemt-config/
    `-- telemt.toml
```

## Requirements

- Debian or Ubuntu-based server
- `bash`
- `apt`
- Internet access to download Docker and the MTProxy image

## Firewall Notes

The script attempts to open the selected TCP port automatically:

- via `ufw` if it is enabled
- otherwise via `iptables` if available

If your cloud provider uses an external firewall or security group, you may still need to allow the port manually.

## Useful Commands

View logs:

```bash
cd /opt/telemt
docker compose logs --tail=50
```

Restart the proxy:

```bash
cd /opt/telemt
docker compose restart
```

Stop the proxy:

```bash
cd /opt/telemt
docker compose down
```

Start it again:

```bash
cd /opt/telemt
docker compose up -d
```

## Notes

- The script uses the `whn0thacked/telemt-docker:latest` image
- MTProxy configuration is generated automatically
- IPv6 is disabled by default in the generated config
- The installer is intended for fresh, simple deployments

## Credits

This setup is based on the `telemt` project:

https://github.com/telemt/telemt
