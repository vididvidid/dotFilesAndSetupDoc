# Setting Up `browser-use` on WSL

This guide provides step-by-step instructions for setting up the `browser-use` development environment on the Windows Subsystem for Linux (WSL). This process includes the crucial steps for enabling GUI application support, which is necessary to run the browser.

-----

### \#\# Step 1: Initial Project Setup

First, clone the repository and set up the Python environment using `uv`.

```bash
# Clone the project and navigate into the directory
git clone https://github.com/browser-use/browser-use
cd browser-use

# Install all dependencies using uv
# This installs uv, creates a venv, and installs packages
./bin/setup.sh

# Create your local environment file from the example
cp .env.example .env
```

-----

### \#\# Step 2: Configure GUI Support for WSL

WSL is a command-line environment by default. To run a graphical application like Google Chrome, you must enable GUI support.

#### **A. Check Your WSL Version**

Open a **Windows PowerShell** terminal and run:

```powershell
wsl -l -v
```

Note whether your Linux distribution is running on `VERSION` **1** or **2**, and follow the correct instructions below.

#### **B. Instructions for WSL 2 (Recommended)**

Modern versions of Windows have built-in GUI support called WSLg. If it's not working, you just need to update it.

1.  **Run PowerShell as Administrator**.
2.  Update WSL and restart it:
    ```powershell
    wsl --update
    wsl --shutdown
    ```
3.  Re-open your WSL terminal (e.g., Kali, Ubuntu) for the next steps.

#### **C. Instructions for WSL 1 (Manual)**

WSL 1 requires a third-party X Server on Windows.

1.  **Install VcXsrv on Windows:** Download and install it from [SourceForge](https://sourceforge.net/projects/vcxsrv/).
2.  **Launch VcXsrv on Windows:** Find "XLaunch" in your Start Menu. Configure it with these settings:
      * `Multiple windows` -\> Next
      * `Start no client` -\> Next
      * **Check the box for `Disable access control`** -\> Next & Finish.
3.  **Set the DISPLAY variable in WSL:** In your WSL terminal, run the following command. This tells Linux where to send the display.
    ```bash
    export DISPLAY=$(grep nameserver /etc/resolv.conf | awk '{print $2}'):0.0
    ```
    *Tip: Add this line to the end of your `~/.bashrc` or `~/.zshrc` file to set it automatically every time you open a new terminal.*

#### **D. Verify GUI Setup**

Let's test the GUI connection with a simple app.

```bash
# Install the x11 test apps
sudo apt update
sudo apt install x11-apps -y

# Run the test
xeyes
```

A small window with a pair of eyes should appear on your Windows desktop. **Do not proceed until this step works.**

-----

### \#\# Step 3: Install Google Chrome & Dependencies

Next, install Google Chrome and its required libraries inside WSL.

```bash
# Download the official .deb package
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

# Install the package and its dependencies
sudo apt install ./google-chrome-stable_current_amd64.deb -y

# Install additional required libraries for browser automation
sudo apt install -yq libgconf-2-4 libnss3 libasound2 libxtst6 libxss1 libgtk-3-0 libgbm-dev libxshmfence-dev

# Clean up the installer
rm google-chrome-stable_current_amd64.deb
```

-----

### \#\# Step 4: Run the Example Script

Now that the environment is fully configured, you can run the example script.

```bash
uv run examples/simple.py
```

This should now launch a Chrome window and execute the script successfully.

-----

### \#\# Troubleshooting Common Issues

  - **Error:** The script fails with a `TimeoutError` or `ConnectionRefusedError`.

      - **Cause:** The browser failed to launch.
      - **Solution:** The GUI setup is likely incorrect. Go back to **Step 2D** and ensure the `xeyes` command works. If it doesn't, carefully review the instructions for your WSL version.

  - **Error:** You run `google-chrome` from the terminal and it fails, even though `xeyes` works.

      - **Cause:** Chrome has specific sandbox or GPU issues within WSL.
      - **Solution:** Test it with flags that fix common WSL problems. If this command works, your setup is correct, and the `browser-use` library should run fine.
        ```bash
        google-chrome --no-sandbox --disable-gpu
        ```
