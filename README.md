# <img src="https://upload.wikimedia.org/wikipedia/commons/1/10/2023_Obsidian_logo.svg" width="2.7%" /> Obsync: A Two-Way Obsidian Sync

Easily sync your Obsidian vault between your desktop and mobile device without purchasing Obsidian Sync. This script leverages FTP for file transfer and ensures data integrity with backup capabilities.

### Key Features:
1. Two-Way Sync: Sync files and folders between desktop and mobile to keep your vault updated on both devices.
2. Exclusion Rules: Customize which files and folders to exclude from the sync to avoid unnecessary data transfer.
3. Automated Backup: Create local backups on your Android device before syncing to prevent data loss.
---
## Before Reading:
> [!Important]
> This guide focuses on **Windows** and **Android**, but the steps can be adapted for other operating systems as well.

> [!Caution]
> Test this script with a mock vault first to ensure the safety of your data. This precaution helps prevent any accidental loss or corruption of your actual vault. Once you have confirmed that everything works as intended, you can proceed to use your main vault.

> [!TIP]
> Got a suggestion? Comment below!
---
## Requirements
- **FTP Server running on Desktop** <sup>[How to set up an FTP server on Windows?](https://www.windowscentral.com/how-set-and-manage-ftp-server-windows-10)</sup>
  > Ensure the Obsidian folder on your desktop is accessible through FTP from your mobile beforehand.
  
- **A Terminal in Mobile** (preferably [Termux](https://github.com/KitsunedFox/termux-monet))

## Installation
1. **Install Required Packages in Termux:**
    ```sh
    pkg update && pkg install lftp rsync
    ```
2. **Create the Sync Script:**
    ```sh
    nano obsidian-sync.sh
    ```
3. **Add the following code to the script:**
    > Don't forget to change the configurations in the `FTP server details` and `Patterns to exclude`.
    
    ```sh
    #!/bin/bash

    # FTP server details
    FTP_HOST="ftp://192.168.1.100:21" # e.g., 192.168.1.100:21
    FTP_USER="user"
    FTP_PASS="password"
    REMOTE_DIR="/Documents/Obsidian" # Obsidian location on Desktop
    LOCAL_DIR="/storage/1BC4-1763/Documents/Obsidian" # Obsidian location on Mobile
    BACKUP_DIR="/storage/1BC4-1763/Documents/Obsidian-Backup" # Backup location on Mobile

    # Patterns to exclude
    EXCLUDE_PATTERNS=(
       # "*.tmp"         # Exclude all .tmp files
       # "*.bak"         # Exclude all .bak files
       # "cache/"        # Exclude cache directory
       # "logs/"         # Exclude logs directory
       # "DoNotSync.md"  # Exclude specific file
       ".obsidian/"      # Exclude Obsidian configs
    )

    # Convert EXCLUDE_PATTERNS to rsync format
    RSYNC_EXCLUDES=()
    for pattern in "${EXCLUDE_PATTERNS[@]}"; do
      RSYNC_EXCLUDES+=(--exclude "$pattern")
    done

    # Convert EXCLUDE_PATTERNS to lftp format
    LFTP_EXCLUDES=""
    for pattern in "${EXCLUDE_PATTERNS[@]}"; do
      LFTP_EXCLUDES+="--exclude $pattern "
    done

    # Function to create a backup using rsync
    backup_files() {
      rsync -av --delete "${RSYNC_EXCLUDES[@]}" "$LOCAL_DIR/" "$BACKUP_DIR/"
    }

    # Sync function using lftp
    sync_files() {
      lftp -u "$FTP_USER","$FTP_PASS" "$FTP_HOST" <<EOF
      set ftp:ssl-allow no
      mirror --verbose --delete --only-newer --no-perms $LFTP_EXCLUDES "$REMOTE_DIR" "$LOCAL_DIR"
      mirror --verbose --reverse --delete --only-newer --no-perms $LFTP_EXCLUDES "$LOCAL_DIR" "$REMOTE_DIR"
      quit
    EOF
    }

    # Create a backup before syncing
    backup_files

    # Call sync function
    sync_files
    ```
4. **Make the Script Executable:**
    ```sh
    chmod +x obsidian-sync.sh
    ```
5. **Run the Script:**
    ```sh
    ./obsidian-sync.sh
    ```
    
---

## Automation
Automating the sync process ensures that your Obsidian vault stays up-to-date without manual intervention. Here are two methods to automate the script:

### Method 1: Using Cron in Termux

You can set up a cron job in Termux to run the script at specified intervals. Follow these steps:

1. **Install Cron Package**:
    ```sh
    pkg install cronie
    ```

2. **Start the Cron Daemon**:
    ```sh
    crond
    ```

3. **Edit the Cron Table**:
    ```sh
    crontab -e
    ```

4. **Add a Cron Job**:
    Add the following line to run the sync script every hour (adjust the schedule as needed):
    ```sh
    0 * * * * /data/data/com.termux/files/home/obsidian-sync.sh
    ```

    This line means "run the script at the start of every hour."

5. **Save and Exit**:
    Save the changes and exit the editor. Your script will now run automatically based on the specified schedule.
    
    > Make sure to keep Termux running in background.

### Method 2: Using Automate/Tasker with Termux:Tasker Plugin

For a more flexible approach, you can use automation apps like Automate or Tasker along with the Termux:Tasker plugin to run the script based on specific events or conditions.

#### Using Automate:
1. **Install Automate and Termux**:
    Download and install [Automate](https://play.google.com/store/apps/details?id=com.llamalab.automate) from the Play Store and the [Termux:Tasker](https://github.com/KitsunedFox/termux-tasker?tab=readme-ov-file#plugin-configuration) plugin from the Play Store.

2. **Create a New Flow in Automate**:
    Open Automate and create a new flow.

3. **Add a Plug-in action Block**:
    - Add a "Plug-in action" block to the flow.
    - In the "Pick plug-in" field, select Termux.
    - This will open a new window.
    - In executable field, place your `/path/to/obsidian-sync.sh`.
      - You can put your `obsidian-sync.sh` in `/.termux/tasker` to avoid any issue.
    - Save and exit, both tasker plugin and the flow.

4. **Add a Trigger Block**:
    Add a trigger block (like "Time" for scheduled sync or "Wi-Fi Connected" for event-based sync) and connect it to Plug-in action block.
    > I'll create a flow for this later, I'm tired now.
5. **Save and Enable the Flow**:
    Save your flow and enable it. The script will now run automatically based on your defined trigger.

#### Using Tasker:
1. **Install Tasker and Termux:Tasker Plugin**:
    Download and install [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) and the [Termux:Tasker](https://github.com/KitsunedFox/termux-tasker?tab=readme-ov-file#plugin-configuration) plugin from the Play Store.

2. **Create a New Profile in Tasker**:
    Open Tasker and create a new profile.

3. **Set a Trigger**:
    - Choose a trigger (like "Time" for scheduled sync or "Wi-Fi Connected" for event-based sync).

4. **Add a Task**:
    - Create a new task and add an action.
    - Select "Plugin" -> "Termux:Tasker".
    - Tap the configuration icon and enter the path to your script:
      ```sh
      /data/data/com.termux/files/home/obsidian-sync.sh
      ```
      - You can put your `obsidian-sync.sh` in `/.termux/tasker` to avoid any issue.

5. **Save and Enable the Profile**:
    Save your profile and enable it. The script will now run automatically based on the defined trigger.

By setting up automation, you ensure that your Obsidian vault syncs regularly without manual intervention. Choose the method that best fits your needs and enjoy seamless synchronization across your devices.


Hope this helps! 

---

## What's Next?
- Add some picture for better understanding.
- A Video Guide.
- A Automate flow for Obsync.

---

Also don't hesitate to comment below for potential improvements or any flaws in the code. Or improve the gist formatting anyway.

Thank you 💖