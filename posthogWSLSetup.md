# My Complete PostHog Setup Notes for WSL (Ubuntu)

This document is a comprehensive guide for setting up the PostHog development environment on WSL (Ubuntu). It covers the recommended WSL configuration, the full installation process, how to run the application, common problems, and their solutions.

-----

## Recommended Environment: WSL2 Configuration

The PostHog development environment, with its multiple services and databases, is resource-intensive. A traditional virtual machine (like VirtualBox) may struggle if it can't be allocated significant and dedicated RAM and CPU cores.

**WSL2 (Windows Subsystem for Linux 2) is the recommended approach** because it integrates more deeply with Windows, allowing for better resource sharing and performance. This setup provides direct access to more CPU cores and generous RAM allocation while running seamlessly alongside your Windows applications.

### The `.wslconfig` File

To ensure WSL has enough resources, create a file named `.wslconfig` in your Windows user profile folder (e.g., `C:\Users\YourUsername\.wslconfig`). Add the following content to it:

```ini
# This file is located at C:\Users\<YourUsername>\.wslconfig
[wsl2]
memory=8GB              # Enough for Docker/PostHog while leaving RAM for Windows
swap=6GB                # More generous swap for safety on heavy builds
localhostForwarding=true    # Access Linux services from Windows at localhost
nestedVirtualization=true   # May be needed for advanced Docker features
dnsTunneling=true           # Fixes potential DNS resolution/networking issues
```

**Note:** After creating or changing this file, you must restart WSL for the settings to apply. Open a PowerShell terminal and run `wsl --shutdown`. The new settings will be active the next time you start your WSL instance.

-----

## Complete Installation Process

This is the definitive step-by-step guide for a fresh installation.

### Step 1: System Prep & Prerequisites

Update the system and install essential tools.

```bash
# Refresh software sources and upgrade all packages
sudo apt update && sudo apt upgrade -y

# Install essential tools for compiling, downloading, and version control
sudo apt install -y build-essential curl git wget
```

### Step 2: Install Python & Rust Tools

Install `ruff` via `pipx` to avoid system conflicts, and install the Rust toolchain.

```bash
# Install pipx, a safe tool for Python CLI apps
sudo apt install -y pipx
pipx ensurepath

# Install the ruff linter
pipx install ruff

# Install the Rust toolchain manager 'rustup'
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup default stable
```

### Step 3: Install Flox

Install the Flox development environment manager by downloading the `.deb` package.

```bash
# Download the .deb package from the official source
wget https://flox.dev/downloads/debian-archive/flox.x86_64-linux.deb -O flox.deb

# Install the package
sudo dpkg -i flox.deb
```

### Step 4: Install Docker & Compose (The Official Way)

Install the complete Docker suite from Docker's official repository to get all the necessary tools.

```bash
# 1. Set up Docker's official repository
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 2. Install the Docker engine, CLI, and Compose plugin
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 5: Configure Docker User Permissions

Add your user to the `docker` group to run commands without `sudo`.

```bash
sudo usermod -aG docker $USER
```

**🚨 Important:** You must **close and reopen your WSL terminal** for this change to take effect.

-----

## Running the Application

### Understanding the Ports: The "8010 Trick"

The PostHog dev environment uses multiple services on different ports, but a proxy manages everything for you. You only need to remember one URL.

  * **`http://localhost:8010` (The Main Gateway)**: **This is the ONLY address you should open in your browser.** A proxy server listens here and intelligently routes your requests to the correct service. It's the main entrance to the application.
  * `http://localhost:8000` (The Backend API): The Python/Django backend runs here. The `8010` gateway sends all API requests to this service.
  * `http://localhost:8234` (The Frontend Dev Server): The JavaScript/React frontend runs here. The `8010` gateway serves the initial HTML page, which then fetches all its assets (JS, CSS) from this server.

### Method 1: Running the Full Stack

This is the standard method to run all PostHog services.

```bash
# Navigate to the project root
cd ~/posthog

# Activate the development environment
flox activate

# Start the Docker daemon if it's not running
sudo service docker start

# Start all services
bin/start
```

Look at the process list in your terminal. Once all key services show a green **`UP`** status, the application is ready at `http://localhost:8010`.

### Method 2: Running a Minimal Stack (Easier on your System)

You don't need to run everything at once. For most frontend or backend work, a minimal set of services is enough and uses fewer system resources.

```bash
# Navigate to the project root and activate the environment
cd ~/posthog
flox activate

# Start the Docker daemon
sudo service docker start

# Start only the essential services
bin/start backend frontend plugin-server celery-worker
```

This will only launch the services you listed, making for a faster and lighter experience. The application will still be available at `http://localhost:8010`.

-----

## Troubleshooting Guide

A quick reference for all the problems we solved.

  * **Problem:** `docker compose` command not found or `E: Unable to locate package docker-compose-plugin`.

      * **Solution:** Your system has the wrong version of Docker. Follow **Step 4** of this guide to install Docker from the official repository.

  * **Problem:** `Cannot connect to the Docker daemon at unix:///var/run/docker.sock`.

      * **Solution:** The Docker background service isn't running. Start it manually with `sudo service docker start`.

  * **Problem:** The `frontend` service shows as `DOWN` or `EXITED` when running `bin/start`.

      * **Solution:** A script is failing. To see the specific error, try to start the frontend manually: `cd frontend && pnpm start`. This will show you the real error message.

  * **Problem:** The browser shows raw code like `{% include "head.html" %}`.

      * **Solution:** You are on the wrong port. You've likely connected directly to the frontend asset server (e.g., `:8234`). **Always use `http://localhost:8010`**, which is the main gateway that assembles the full page.
