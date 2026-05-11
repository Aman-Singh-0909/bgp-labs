# BGP Security Labs 🌐

A hands-on collection of BGP security labs built on Kali Linux using Docker and Containerlab.

## Labs

| Lab | Topic | Status |
|-----|-------|--------|
| Lab 1 | BGP Hijack Simulation | ✅ Complete |
| Lab 2 | RPKI Validation with FRRouting | 🚧 Coming Soon |
| Lab 3 | Real BGP Analysis with bgp.he.net | 🚧 Coming Soon |

## Requirements

- Kali Linux
- Docker
- Containerlab
- FRRouting (FRR)

## Quick Start

```bash
# Install containerlab
sudo bash -c "$(curl -sL https://get.containerlab.dev)"

# Deploy Lab 1
sudo containerlab deploy -t lab1-bgp-hijack/bgp-hijack.yml
```

## Author
Aman-Singh
