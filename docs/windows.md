# Windows 10/11 Installation Guide

This guide will help you set up a local Symfony development environment on Windows using WSL2 (Windows Subsystem for Linux).

> **Note:** Symfony applications work best with UNIX-based systems. This guide uses WSL2 to provide a native Linux environment on Windows, ensuring optimal performance and compatibility.

## Prerequisites

### 1. Install WSL2

Open Command Prompt or PowerShell as Administrator and run:

```cmd
wsl --version
```

**Ensure your WSL version is 0.67.6 or higher.** If WSL is not installed or needs updating, follow the [official Microsoft guide](https://learn.microsoft.com/en-us/windows/wsl/install).

### 2. Install Ubuntu in WSL

```cmd
wsl --install Ubuntu
```

This will install Ubuntu as a virtual machine within WSL.

### 3. Enable systemd

After Ubuntu is installed, enter WSL and configure systemd:

```bash
sudo nano /etc/wsl.conf
```

Add the following content:

```ini
[boot]
systemd=true
```

Save and exit (`Ctrl + X`, then `Y`, then `Enter`).

Restart the Ubuntu VM from Command Prompt:

```cmd
wsl --terminate Ubuntu
```

Then start Ubuntu again from the Start menu.

### 4. Install Homebrew in WSL

Inside the Ubuntu terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to your PATH:

```bash
(echo; echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"') >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew --version
```

### 5. Install Build Tools

The Symfony CLI and some packages require build tools:

```bash
sudo apt-get update
sudo apt-get install -y gcc make
gcc --version
```

### 6. Install Docker

Install Docker Engine directly inside WSL:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo gpasswd -a $USER docker
rm get-docker.sh
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Log out and back in, or run `newgrp docker` for the group changes to take effect.

#### Alternatives (Not Tested)

> **Note:** The following alternatives may work but have not been tested with this guide. Instructions may be incomplete.

- **[Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)** - GUI application with WSL2 backend integration (licensing may apply)

## Setting Up Your Project

### Important: Use the Linux Filesystem

For optimal performance, **always work within the Linux filesystem**, not the mounted Windows drives.

```bash
# Good - Fast performance
cd ~/projects/my-symfony-app

# Bad - Slow performance
cd /mnt/c/Users/YourName/projects/my-symfony-app
```

Create a workspace directory:

```bash
mkdir -p ~/projects
cd ~/projects
```

### Clone Your Project

```bash
git clone <your-repository-url>
cd your-project-name
```

### Copy Template Files

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

> **Tip:** You can access your WSL files from Windows Explorer at `\\wsl$\Ubuntu\home\<username>\`.

### Install Dependencies

```bash
make install
```

This command will:
- Execute `brew bundle` to install all dependencies from the Brewfile
- Install [PHP](https://github.com/shivammathur/homebrew-php) with necessary extensions
- Install [Composer](https://getcomposer.org/)
- Install the [Symfony CLI](https://symfony.com/download)

### Import SSL Certificate to Windows

To access your local HTTPS server from Windows browsers, import the Symfony certificate:

1. Open the certificate folder. Either run this from WSL:
   ```bash
   explorer.exe `wslpath -w $HOME/.symfony5/certs`
   ```
   Or navigate manually in Windows Explorer to `\\wsl$\Ubuntu\home\<username>\.symfony5\certs\`
2. Double-click `default.p12`
3. Follow the Certificate Import Wizard
4. Install to "Local Machine" > "Trusted Root Certification Authorities"

### Start the Development Environment

```bash
make run
```

When complete, you should see:

```
[OK] Web server listening
     The Web server is using PHP FPM 8.x
     https://127.0.0.1:8000
```

Access your app in Windows at `https://localhost:8000`.

## Optional: Custom Domain Setup

### Configure Windows Proxy

1. Open **Settings** > **Network & Internet** > **Proxy**
2. Enable **Use setup script**
3. Set Script address to: `http://localhost:7080/proxy.pac`
4. Click **Save**

For more details, see [Windows proxy settings guide](https://support.microsoft.com/en-us/windows/use-a-proxy-server-in-windows-03096c53-0554-4ffe-b6ab-8b1deee8dae1).

### Attach Your Project

Inside WSL:

```bash
make proxy-attach DOMAIN=my-app
```

Your app will now be available at `https://my-app.wip`.

## Recommended IDE

**Visual Studio Code** is strongly recommended for WSL development. It has seamless WSL integration:

1. Install [VS Code](https://code.visualstudio.com/) on Windows
2. Install the [Remote - WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
3. Open your project: `code ~/projects/your-project-name`

This provides a native development experience while running code in Linux.

## Troubleshooting

### Temporary Failure in Name Resolution

If you encounter DNS resolution issues, add a public DNS server:

```bash
echo "nameserver 8.8.8.8" | sudo tee -a /etc/resolv.conf
```

### Homebrew Issues

If Homebrew seems stuck or unresponsive:

1. Press `Ctrl + C` to cancel the current process
2. Run `brew doctor` to diagnose issues
3. Follow any recommendations shown
4. Try `make install` again

### Homebrew Binary Not Found

If you see `Command 'brew' not found`:

```bash
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
```

### Docker Permission Denied

```bash
sudo gpasswd -a $USER docker
newgrp docker  # Or log out and back in
```

### Slow Performance

If your project is running slowly:

1. Ensure your project is in the Linux filesystem (`~/`), not `/mnt/c/`
2. Avoid cross-filesystem operations

### WSL Not Starting

If WSL fails to start:

```cmd
wsl --shutdown
wsl
```

### Port Already in Use

```bash
symfony server:stop --all  # Stop all Symfony servers

# If the above doesn't work, find and kill the process manually
lsof -i :8000              # Find the process using the port
kill -9 <PID>              # Kill the process (replace <PID> with actual ID)

make run                   # Try again
```

---

[Back to main guide](../README.md)
