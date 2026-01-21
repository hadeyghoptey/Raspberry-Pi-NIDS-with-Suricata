# Raspberry Pi NIDS with Suricata + GitHub Alert Bot

A lightweight Network Intrusion Detection System (NIDS) for Raspberry Pi using Suricata with automated GitHub-based alert notifications.

## Features

- **Lightweight**: Designed for low-resource hardware
- **CLI-based**: No GUI required
- **Comprehensive detection**: Alerts on port scanning, intrusion attempts, and suspicious network activity
- **Automated alerts**: Pushes Suricata alerts to GitHub Issues
- **Easy setup**: Simple installation and configuration

## GitHub Integration

The included Python bot automatically creates GitHub Issues for detected alerts:

- Creates a private repository for alerts
- Uses GitHub Personal Access Token for authentication
- Monitors Suricata logs in real-time
- Can run as a systemd service

## Requirements

- Raspberry Pi of any model
- Network interface (Ethernet or Wi-Fi)
- Python 3

## Documentation

See `Guide.md` for detailed installation and configuration instructions.

## Disclaimer

For educational and defensive security purposes only. Only deploy on networks you own or have permission to monitor.