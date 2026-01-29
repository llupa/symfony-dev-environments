# Ubuntu/Linux Installation Guide

This guide will help you set up a local Symfony development environment on Ubuntu or other Debian-based Linux distributions.

## Prerequisites

### 1. Install Homebrew

[Homebrew](https://brew.sh/) works on Linux and provides a consistent way to manage packages across platforms.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to your PATH:

```bash
# Add to your shell profile
(echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# Verify installation
brew --version
```

> **Note:** On Linux, Homebrew creates a `linuxbrew` user. Learn more in the [Homebrew on Linux documentation](https://docs.brew.sh/Homebrew-on-Linux#install).

### 2. Install Docker

Install Docker Engine using the convenience script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo gpasswd -a $USER docker
rm get-docker.sh
```

Enable Docker services to start on boot:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

> **Important:** Log out and log back in for the group changes to take effect, or run `newgrp docker`.

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

### Configure System Proxy

For Ubuntu with GNOME:

1. Open **Settings** > **Network**
2. Click the gear icon next to your connection
3. Go to **Network Proxy**
4. Select **Automatic**
5. Set the Configuration URL to: `http://127.0.0.1:7080/proxy.pac`

For more details, see [Ubuntu's proxy documentation](https://help.ubuntu.com/stable/ubuntu-help/net-proxy.html.en).

### Attach Your Project

```bash
make proxy-attach DOMAIN=my-app
```

Your app will now be available at `https://my-app.wip`.

## Troubleshooting

### Temporary Failure in Name Resolution

If you encounter DNS resolution issues, add a public DNS server:

```bash
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
```

For a permanent fix, configure your network manager or systemd-resolved.

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
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
```

### Docker Permission Denied

If you get permission denied errors with Docker:

```bash
sudo gpasswd -a $USER docker
newgrp docker  # Or log out and back in
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

---

[Back to main guide](../README.md)
