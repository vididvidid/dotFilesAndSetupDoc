# How to Move a WSL Distribution from C: to D: Drive

This guide provides a complete, step-by-step process for relocating an existing Windows Subsystem for Linux (WSL) distribution from your C: drive to another drive (e.g., D:) to save space.

This process involves exporting your current distro, removing it from its original location, and importing it into the new location.

-----

## \#\# Step 1: Identify and Back Up Your WSL Distro

First, we need to find the name of your distribution and create a safe backup. All commands in this step should be run in **PowerShell**.

1.  **List your installed WSL distributions** to find the exact name.

    ```powershell
    wsl --list -v
    ```

    *(Look for the name of your distro in the `NAME` column. We will use `<DistroName>` as a placeholder for it in the following commands.)*

2.  **Shut down your distro** to ensure no files are in use during the backup.

    ```powershell
    wsl --terminate <DistroName>
    ```

3.  **Create a backup folder** on your D: drive.

    ```powershell
    mkdir D:\wsl-backups
    ```

4.  **Export the distro** to a `.tar` backup file. This may take several minutes depending on the size of your installation.

    ```powershell
    wsl --export <DistroName> D:\wsl-backups\<DistroName>.tar
    ```

-----

## \#\# Step 2: Remove the Old Distro and Import to D: Drive

Now that you have a safe backup, you can remove the original installation and import it to the new location.

1.  **Unregister the old distro from the C: drive.**

    > **⚠️ Warning:** This step will permanently delete the original WSL installation from your C: drive. Proceed only after confirming the export in the previous step was successful.

    ```powershell
    wsl --unregister <DistroName>
    ```

2.  **Create the new home folder** for your WSL installation on the D: drive.

    ```powershell
    mkdir D:\wsl
    ```

3.  **Import the backup** into the new location. This command tells WSL to create a new distro with the same name, storing its files in `D:\wsl\<DistroName>`, using the backup file you created.

    ```powershell
    wsl --import <DistroName> D:\wsl\<DistroName> D:\wsl-backups\<DistroName>.tar --version 2
    ```

-----

## \#\# Step 3: Configure the Default User

After importing, WSL defaults to logging in as the `root` user. You must reconfigure it to use your personal user account.

1.  **Log into your distro as the `root` user.** Run this from PowerShell:

    ```powershell
    wsl -d <DistroName> -u root
    ```

2.  **Create and edit the `wsl.conf` file** using the `nano` text editor. This command is run inside the Linux shell you just opened.

    ```bash
    nano /etc/wsl.conf
    ```

3.  **Add the following configuration** to the file. This tells WSL which user to log in as by default. (Replace `<your-username>` with your actual Linux username).

    ```ini
    [user]
    default = <your-username>
    ```

    To save and exit `nano`:

      * Press **`Ctrl + O`**, then **`Enter`** to save the file.
      * Press **`Ctrl + X`** to exit the editor.

4.  **Shut down WSL completely** to apply the changes. First, exit the Linux shell:

    ```bash
    exit
    ```

    Then, back in **PowerShell**, run the shutdown command:

    ```powershell
    wsl --shutdown
    ```

5.  **Verify the change.** The next time you start your distro, you should be logged in as your correct user.

    ```powershell
    wsl -d <DistroName>
    ```

-----

## \#\# Step 4: Final Cleanup (Optional)

Once you have confirmed that your new WSL installation on the D: drive is working perfectly, you can remove the backup file to save space.

  * **Delete the backup `.tar` file:**
    ```powershell
    rm D:\wsl-backups\<DistroName>.tar
    ```
