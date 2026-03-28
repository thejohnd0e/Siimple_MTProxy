# Siimple MTProxy

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
