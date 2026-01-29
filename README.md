# Symfony Local Development Environments

Guides and templates for running Symfony applications locally across macOS, Linux, and Windows.

## Overview

This repository helps development teams maintain consistency across different operating systems. By adding a `Brewfile`, `Makefile`, and `docker-compose.yaml` to your Symfony project, every team member can set up their local environment with:

```bash
make install  # Install PHP, Composer, Symfony CLI
make run      # Start Docker services and web server
```

Same commands. Same PHP version. Same services. Any OS.

## What's Included

- **Brewfile template** - Homebrew dependencies for PHP, Composer, and Symfony CLI
- **Makefile template** - Common automation commands for installation and running your app
- **docker-compose.yaml template** - Container services setup (PostgreSQL, RabbitMQ, etc.)
- **OS-specific guides** - Step-by-step installation instructions for each platform

## Choose Your Operating System

| OS | Guide | Notes |
|---|---|---|
| macOS | [Installation Guide](docs/macos.md) | Uses Homebrew + Docker Desktop/Rancher Desktop |
| Ubuntu/Linux | [Installation Guide](docs/ubuntu.md) | Uses Homebrew + Docker Engine |
| Windows 10/11 | [Installation Guide](docs/windows.md) | Uses WSL2 + Ubuntu + Docker |

## Templates

Ready-to-use templates to bootstrap your Symfony project:

- [Brewfile](templates/Brewfile) - Homebrew dependencies
- [Makefile](templates/Makefile) - Automation commands
- [docker-compose.yaml](templates/docker-compose.yaml) - Container services (PostgreSQL, RabbitMQ)

## Quick Start

1. Choose your OS guide from the table above
2. Copy the templates to your Symfony project root
3. Run `make install` to install dependencies
4. Run `make run` to start your development environment

## Prerequisites

All platforms require:

- [Homebrew](https://brew.sh/) (package manager)
- [Docker](https://www.docker.com/) or [Rancher Desktop](https://rancherdesktop.io/) (container runtime)
- Git (version control)

## Project Structure

```
your-symfony-project/
├── Brewfile              # Homebrew dependencies
├── Makefile              # Automation commands
├── docker-compose.yaml   # Container services (databases, cache, etc.)
├── composer.json         # PHP dependencies
└── ...
```

## Support

If you encounter issues, check the troubleshooting section in each OS-specific guide.

## Contributing

This guide is based on information gathered from colleagues across different platforms. I can only personally verify the macOS setup, so contributions and troubleshooting help for Linux and Windows (WSL) are especially welcome.

Feel free to open issues or submit pull requests to improve these guides.
