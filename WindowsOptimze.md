# Windows Optimization for Development

This document outlines a series of optimizations to configure a Windows 11 machine for a minimal, high-performance development environment.

## 1. Disable Startup Applications

**Goal:** Prevent unnecessary applications from launching at startup to speed up boot time and reduce background resource usage.

**Steps:**
1.  Press `Ctrl + Shift + Esc` to open Task Manager.
2.  Navigate to the **"Startup apps"** tab.
3.  Right-click on any non-essential application (e.g., Free Download Manager, OneNote, etc.) and select **"Disable"**.
4.  Pay attention to the "Startup impact" column to prioritize disabling high-impact apps.

---

## 2. Completely Remove OneDrive

**Goal:** Prevent files from saving to OneDrive and remove the application and its integration from the system.

**Steps:**

**A. Unlink and Stop Backup:**
1.  Open OneDrive settings from the system tray icon (⚙️ > Settings).
2.  Under **"Sync and backup"**, click **"Manage backup"** and "Stop backup" for all folders (Desktop, Documents, Pictures).
3.  Under the **"Account"** tab, click **"Unlink this PC"**.

**B. Uninstall the App:**
1.  Go to **"Add or remove programs"**.
2.  Search for "Microsoft OneDrive" and uninstall it.

**C. (Optional) Clean Up Leftovers:**
1.  Delete the empty `C:\Users\<YourUsername>\OneDrive` folder.
2.  To remove the File Explorer icon, open Registry Editor (`regedit`) and delete the key: `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Desktop\NameSpace\{018D5C66-4533-4307-9B53-224DE2ED1FE6}`

---

## 3. Optimize File Explorer

**Goal:** Prevent File Explorer from automatically starting WSL and improve its responsiveness.

**Steps:**
1.  Open File Explorer, click the `...` menu, and select **Options**.
2.  Change **"Open File Explorer to:"** from "Home" to **"This PC"**.
3.  Click Apply and OK.

---

## 4. Disable System Animations

**Goal:** Make the user interface feel faster and more responsive by removing visual effects.

**Steps:**
1.  Press `Win + R`, type `sysdm.cpl`, and press Enter.
2.  Go to the **"Advanced"** tab and click **"Settings..."** under Performance.
3.  Select **"Adjust for best performance"**.
4.  **(Recommended)** Re-check the boxes for:
    * `Show thumbnails instead of icons`
    * `Smooth edges of screen fonts`
5.  Click Apply and OK.

---

## 5. System Cleanup

**Goal:** Free up disk space by removing unnecessary and corrupt system files.

**Steps:**
1.  Go to **Settings > System > Storage**.
2.  Click on **Cleanup recommendations**.
3.  Select and remove **"Previous Windows installation(s)"**. These are leftover files from old updates and are often large and useless.
4.  It is also safe to remove **"System recovery log files"** if your PC is running without issues.
