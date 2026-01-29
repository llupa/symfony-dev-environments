# macOS Installation Guide

This guide will help you set up a local Symfony development environment on macOS.

## Prerequisites

### 1. Install Homebrew

[Homebrew](https://brew.sh/) is the package manager for macOS. Install it by running:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the post-installation instructions shown in your terminal to add Homebrew to your PATH.

### 2. Install Docker

You need a container runtime to run services like databases and caches. Choose one of the following options:

#### Option A: Docker Desktop (Recommended)

Install [Docker Desktop for Mac](https://docs.docker.com/desktop/install/mac-install/).

1. Download and install Docker Desktop
2. Start Docker Desktop from Applications
3. Verify installation: `docker --version`

#### Option B: Rancher Desktop (Free Alternative)

If you cannot use Docker Desktop due to licensing requirements, [Rancher Desktop](https://docs.rancherdesktop.io/getting-started/installation/) is a free alternative.

> **Note:** If switching from Docker Desktop, run `docker system prune --all --volumes` and uninstall Docker Desktop before installing Rancher Desktop.

**Important:** Configure Rancher Desktop to use the Docker engine:

1. Open Rancher Desktop
2. Click on Preferences (gear icon)
3. Navigate to Container Engine > General
4. Select `dockerd (moby)`
5. Apply changes and wait for Rancher Desktop to reload

## Installation

### 1. Copy Template Files

Copy the following templates to your Symfony project root:

- [Brewfile](../templates/Brewfile) - Homebrew dependencies
- [Makefile](../templates/Makefile) - Automation commands
- [docker-compose.yaml](../templates/docker-compose.yaml) - Container services

**Important:** Review and customize the templates according to your project needs. The templates work well for typical Symfony applications, but you may need to:

- Adjust the PHP version in the Brewfile (e.g., `php@8.4` instead of `php@8.5`)
- Enable/disable PHP extensions your project requires
- Remove services you don't need (e.g., RabbitMQ if not using async messaging)
- Add additional services (e.g., Redis, Elasticsearch)

For project-specific Symfony CLI configuration (e.g., `.php-version`, `.symfony.local.yaml`), see the [Symfony CLI documentation](https://symfony.com/doc/current/setup/symfony_cli.html).

### 2. Install Dependencies

Run the following command in your project directory:

```bash
make install
```

This command will:
- Execute `brew bundle` to install all dependencies from the Brewfile
- Install [PHP](https://github.com/shivammathur/homebrew-php) with necessary extensions
- Install [Composer](https://getcomposer.org/)
- Install the [Symfony CLI](https://symfony.com/download)
- Start the Symfony proxy server

After installation, you should have a running proxy at `https://127.0.0.1:7080`.

### 3. Start the Development Environment

```bash
make run
```

This will:
- Start Docker containers defined in your `docker-compose.yaml`
- Start the Symfony local web server

When complete, you should see:

```
[OK] Web server listening
     The Web server is using PHP FPM 8.x
     https://127.0.0.1:8000
```

## Optional: Custom Domain Setup

You can configure a custom local domain (e.g., `https://my-app.wip`) using the Symfony proxy.

### 1. Configure System Proxy

1. Open **System Preferences** > **Network**
2. Select your active network connection
3. Click **Advanced** > **Proxies**
4. Enable **Automatic Proxy Configuration**
5. Set the URL to: `http://127.0.0.1:7080/proxy.pac`
6. Click **OK** and **Apply**

For more details, see [Apple's proxy settings guide](https://support.apple.com/guide/mac-help/change-proxy-settings-on-mac-mchlp2591/mac).

### 2. Attach Your Project

```bash
make proxy-attach DOMAIN=my-app
```

Your app will now be available at `https://my-app.wip`.

## Troubleshooting

### Homebrew Issues

If Homebrew seems stuck or unresponsive:

1. Press `Ctrl + C` to cancel the current process
2. Run `brew doctor` to diagnose issues
3. Follow any recommendations shown
4. Try `make install` again

### Homebrew Binary Not Found

If you see `Command 'brew' not found`:

1. Check the Homebrew installation output for PATH instructions
2. Add Homebrew to your shell configuration:

```bash
# For Zsh (default on macOS)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc

# For Bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bash_profile
source ~/.bash_profile
```

### Port Already in Use

If port 8000 is already in use:

```bash
symfony server:stop --all  # Stop all Symfony servers

# If the above doesn't work, find and kill the process manually
lsof -i :8000              # Find the process using the port
kill -9 <PID>              # Kill the process (replace <PID> with actual ID)

make run                   # Try again
```

### Docker Issues

If Docker commands fail:

1. Ensure Docker Desktop or Rancher Desktop is running
2. If using Rancher Desktop, verify Docker engine is configured (not containerd)
3. Run `docker ps` to test the connection

---

[Back to main guide](../README.md)
