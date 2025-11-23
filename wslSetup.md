
# The "Bulletproof" Guide to Setting Up i3wm on WSL2

Most guides for setting up the i3 Window Manager on Windows Subsystem for Linux (WSL) fail because they do not account for **Windows Firewall**, **Anti-Virus interference**, or the new **WSL Mirrored Networking** mode.

This guide is the result of extensive troubleshooting to create a stable, error-free tiling window manager environment on Windows.

## 📋 Prerequisites

1.  **WSL2 Installed** (Kali, Ubuntu, etc.)
2.  **Windows 10 or 11**
3.  **VcXsrv** (Windows X Server) installer.

---

## 🚀 Step 1: Clean & Install Dependencies

Open your WSL terminal and ensure your system is fresh. We install `xterm` as a backup terminal and `network-manager-gnome` to prevent status bar errors.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install i3 i3status dmenu i3lock xterm network-manager-gnome -y
````

*Optional (Better Terminal):*

```bash
sudo apt install alacritty
```

-----

## 🖥️ Step 2: Install & Configure the X Server (Windows Side)

i3 requires a display server to draw windows. We use **VcXsrv**.

1.  Download and install [VcXsrv](https://sourceforge.net/projects/vcxsrv/).
2.  Launch **XLaunch** from the Windows Start Menu.
3.  **Display Settings:**
      * Select **"One large window"**.
      * Display number: `0`
4.  **Client Startup:**
      * Select **"Start no client"**.
5.  **Extra Settings (CRITICAL):**
      * ❌ **Uncheck** "Native opengl" (Prevents crashing/black screens).
      * ✅ **Check** "Disable access control" (Vital: allows WSL to talk to Windows).
      * **Additional parameters:** Type `-ac` in the text box.
6.  **Finish:** Click "Save configuration" to your Desktop so you can launch it with one click next time.

-----

## 🛡️ Step 3: The Firewall Fix (The Common Failure Point)

The number one reason for "Cannot Open Display" is the firewall blocking port 6000.

### Option A: PowerShell Command (Recommended)

Open PowerShell as **Administrator** and run this to open the port:

```powershell
New-NetFirewallRule -DisplayName "WSL VcXsrv Inbound" -Direction Inbound -LocalPort 6000 -Action Allow -Protocol TCP -Profile Any
```

### Option B: Third-Party Anti-Virus (McAfee, Norton, etc.)

**⚠️ Important:** If you use McAfee or similar AV, the PowerShell command above will be ignored.

1.  Go to your AV Firewall Settings.
2.  **Whitelist** `vcxsrv.exe` (Allow Incoming/Outgoing).
3.  **Open Port 6000** manually in the AV settings.
4.  *Extreme Case:* If it still fails, uninstall the AV and use the manufacturer's removal tool (e.g., `MCPR` for McAfee) to clear hidden drivers.

-----

## ⚙️ Step 4: Configure WSL (.bashrc)

We need to tell Linux where to send the graphics.

**The Fix for "Mirrored Networking":**
If you are using modern WSL (Mirrored Mode), Linux shares the `localhost` with Windows. The old method of finding the "Nameserver IP" will cause "Connection Refused."

1.  Open your config:

    ```bash
    nano ~/.bashrc
    ```

2.  Add these lines to the very bottom:

    ```bash
    # Connect to VcXsrv on Localhost (For Mirrored Mode/WSL2)
    export DISPLAY=127.0.0.1:0

    # Force indirect rendering to prevent GPU crashes
    export LIBGL_ALWAYS_INDIRECT=1
    ```

3.  Save (`Ctrl+O`, Enter, `Ctrl+X`) and reload:

    ```bash
    source ~/.bashrc
    ```

-----

## 🚀 Step 5: Launching i3

1.  **Start VcXsrv** (Double click your saved XLaunch config). You should see a black window appear.
2.  **In WSL Terminal**, run:
    ```bash
    i3
    ```
3.  **Switch to the VcXsrv Window.**
      * Press **Enter** to generate the config.
      * Press **Enter** to accept the status bar.
      * **CRITICAL:** Press **`Alt`** when asked for the modifier key (Do not use the Windows key, it conflicts with OS shortcuts).

You should now see a small status bar at the bottom of the black screen. **Success.**

-----

## 🐛 Troubleshooting

### Error: `i3: Cannot open display`

  * **Cause:** Linux cannot find the Windows IP.
  * **Fix:** Check `echo $DISPLAY`. If it is blank, redo Step 4. If it shows an IP, ensure VcXsrv is running.

### Error: `Connection refused`

  * **Cause:** VcXsrv is running but rejecting the connection, or Firewall is blocking.
  * **Fix:**
    1.  Ensure "Disable Access Control" is checked in XLaunch.
    2.  Check for third-party Antivirus blocking Port 6000.
    3.  Verify connection with `nc -zv 127.0.0.1 6000`.

### Error: `PID does not belong to any known session` (xss-lock)

  * **Cause:** WSL does not have a true "Login Session" like a real Linux desktop.
  * **Fix:** Ignore this error. It is harmless logs.

### Issue: Mouse cursor is invisible

  * **Fix:** Add `export XCURSOR_THEME=whiteglass` to your `~/.bashrc`.

-----

## ⌨️ Cheat Sheet: Basic Commands

| Action | Shortcut (Assumes Alt is Mod) |
| :--- | :--- |
| **Open Terminal** | `Alt` + `Enter` |
| **Close Window** | `Alt` + `Shift` + `Q` |
| **App Launcher (dmenu)** | `Alt` + `D` |
| **Switch Workspace** | `Alt` + `1` through `9` |
| **Move Window** | `Alt` + `Shift` + `1` through `9` |
| **Split Horizontal** | `Alt` + `H` |
| **Split Vertical** | `Alt` + `V` |
| **Reload Config** | `Alt` + `Shift` + `C` |
| **Exit i3** | `Alt` + `Shift` + `E` |

