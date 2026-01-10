# Setting Up Restic For Windows 11 With Task Scheduler
This repository documents a **modern, minimal, and correct** way to create a __Time-Machine__ style backup service for Windows using Restic.
Actually, this service may be better than Apple's own __Time-Machine__ and best of all it's free.  This guide will cover: 

- **Downloading and installing Restic**
- **Creating a Restic back up repository**
- **Creating all the necessary scripts to deploy a Restic backup service**
- **Scheduling and automating backups using Windows Task Scheduler**

For official Restic documentation click the link below:

[Official Restic Documentation](https://restic.net/)

<br>

---

## Why this guide exists

Let's face it, a lot of back-up software/services are subscription only which is super annoying.
Sometimes you just want to find backup software the just "Works" without any additional hooks or attachments.
Restic does just that.  This guide is for users who don't mind getting their hands a little dirty to implement
a free and open source back-up services of their own.

<br>

---

## Step 1 — Download and install Restic using Chocolatey

If you need to install __Chocolatey__ first, you can find the installation guide at the link below:

[Official Chocolatey Installation Guide](https://chocolatey.org/install)

Launch powershell from your windows environment (Make sure you're in admin mode)

Type in the following command:

```Powershell
choco install restic
```

<br>

>[!Note]
>You might need to restart powershell in admin mode for the install to take effect

<br>

---

## Step 2 — Create and initialize the backup repository

Create a folder which will hold the backups.  In this example i'm using a thumbrdive located at `E:\`
You can use whatever you like for your own combination of drive and folder.

>[!important]
>This guide assumes you are backing up your hard drive to a DIFFERENT drive other than your windows hardrive (e.g. `C:\`).

<br>

```Powershell
New-Item -Itemtype Directory -Path E:\ResticRepo
```

<br>

If you goof up use recursive remove and try again:
```PowerShell
Remove-Item -Recurse -Force E:\ResticRepo
```

<br>

Now initialize the repository folder:
```Powershell
restic init --repo E:\ResticRepo
```
<br>

>[!important]
>You will be asked for a password when you initialize your repository.  Restic uses AES-256 bit encryption to protect your respository so choose a strong password and one you'll remember.
>If you lose the password you will **NOT** be able to access or restore any of your backups so **WRITE IT DOWN** and keep it safe.

<br>

---

## Step 3 — Perform a dry-run to test the respository

Performing a dry-run on the folder(s) we want to back-up allows us to catch any errors before actually backingup any data.
Use the following command to perform a dry run:

```Powershell
restic -r E:\ResticRepo backup D:\My\Path\To\Data C:\My\Other\Path\To\Data --exclude D:\Games --exclude C:\Art\Jpgs --dry-run
```

<br>

>[!Note]
>Notice we can supply multiple sources (drives and letters) to back up and we can also exclude multiple sources as well.  See the offical Restic docs for addtional functionality here

<br>

>[!important]
>If you choose to backup the root of an entire drive (e.g. `H:\`), it's better to place all the contents of what you want to back up into a single folder; for example: `H:\MyData\AllOfYourDriveData`.
>The reason for this is, there are hidden directories in the root of any drive and you may run into _inaccessible_ or _permission denied_
>errors from weirdly hidden directories such as `Recycle` or `System Volume Information` which Restic will not be able to access.  So to avoid any unecessary errors, instead of backing up the root of any drive,
>just place all of your files in a single folder inside of the root drive and select that folder to backup.

<br>

---

## Step 4 — Perform your first backup

If the `--dry-run` succeeds without issue you can then perform your first actual backup manually:

```Powershell
restic -r E:\ResticRepo backup D:\My\Path\To\Data C:\My\Other\Path\To\Data --exclude D:\Games --exclude C:\Art\Jpgs
```
<br>

>[!Note]
>Notice that the command is exactly the same one as in **Step 3** but with the `--dry-run` flag eliminated

<br>

---

## Step 5 — Check your backup

Once your backup is complete you can see your snapshots wth the following command:

```Powershell
restic -r E:\ResticRepo snapshots
```
<br>

Or you can see all of your latest files with this command:

```Powershell
restic -r E:\ResticRepo ls latest
```

<br>

As usual check the official Restic documentation for additional commands

<br>

---

## Step 6 — Restore your files from the latest repo

If you want to restore your files from the latest backup use the following command:

```Powershell
restic -r E:\ResticRepo restore latest --target C:\TempRestore
```
<br>

---


## Step 7 — (Optional) Set the retention policy

You can manage how many daily, weekly, and monthly backup snapshots are kept with the following command:

```Powershell
restic -r E:\ResticRepo forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

<br>
---


## Step 8 — Write Powershell scripts to automate Restic

Now that the repo is setup, we will write a few powershell scripts to automate the backup process:

#### <ins>Script 1 The ENTRY Script</ins>

Create a new folder called `Scripts` in the root of your windows drive.  In my case I will use `C:\Scripts`:

```Powershell
C:\> mkdir Scripts
```

Navigate to your scripts folder:

```Powershell
C:\> cd Scripts
```

If you don't already have it, install `edit` in powershell or you can use any editor you want

```Powershell
C:\Scripts> winget install --id Microsoft.Edit
```

Launch `edit` with the following file name:


```Powershell
C:\Scripts> edit run-restic.ps1
```

Copy and paste the following code into the editor:


```Powershell
# The drive we are backing up to
$backupDrive = "E:\"

# Where the backup script lives, this file is just the entry script
$script = "E:\PowerShellScripts\restic-backup.ps1"

# A log file for debuggging or any troubleshooting
$log = "C:\Scripts\restic-run.log"

"------ $(Get-Date) ------" >> $log

Get-PSDrive >> $log

# If the backup drive volume isn't mounted, do nothing
if (-not (Test-Path $backupDrive)) { 
"Thumbdrive drive not present, exiting." | Out-File -Append $log
exit 0 }

"Thumbdrive drive present, attempting to launch script." | Out-File -Append $log

# Also verify the script which directs the backup process exists
if (-not (Test-Path $script)) { 
"Script not present, exiting." | Out-File -Append $log
exit 0 }

"Script present, attempting to launch restic." | Out-File -Append $log

# Run the real backup script
powershell.exe -NoProfile -ExecutionPolicy Bypass -File $script | Out-File -Append $log
```

<br>

Go to `File` => `Save As` and make sure the file is saved as `run-restic.ps1`.  Then exit

<br>

#### <ins>Script 2 the VB script</ins>

Wait.  What? A Visual Basic (vb) script? Why?  Well the answer is relatively simple.  We want to use Microsoft's Task Scheduler to 
automate calling the `run-restic.ps1` script, but because it's a powershell script the powershell window will pop up briefly during
the times in which the Task Scheduler executes `run-restic.ps1`.  This can be an annoyance if you're working on the desktop and
the powershell window pops up in the background or temporarily steals the focus from your current window.  By having the Task Scheduler
execute a vb script which then calls the `run-restic.ps1` script, we can eliminate the backup process to be a purely silent background service (no powershell pop-ups at all).
In your current directory where the `run-restic.p1` script resides, run the following command:

```Powershell
C:\Scripts> edit run-restic.vbs
```

Then copy and paste the following code:

```Powershell
CreateObject("Wscript.Shell").Run "powershell.exe -NoProfile -ExecutionPolicy Bypass -File ""C:\Scripts\run-restic.ps1""", 0, False
```

<br>

Go to `File` => `Save As` and make sure the file is saved as `run-restic.vbs`.  Then exit

<br>

#### <ins>Script 3 the Backup Process script</ins>

This will be the last powershell script we write. Navigate to your backup drive, for me it's `E:\`, and create
a new folder called `PowerShellScripts`:

```Powershell
E:\ mkdir PowerShellScripts
```

Navigate to the new folder

```Powershell
E:\ cd PowerShellScripts
```

Create a new powershell script `restic-backup.ps1`

```Powershell
E:\PowerShellScripts> edit restic-backup.ps1`
```

Copy and paste the following code:

```Powershell
$ErrorActionPreference = "Stop"
$env:RESTIC_REPOSITORY = "E:\ResticRepo"
$env:RESTIC_PASSWORD_FILE = "E:\PowerShellScripts\restic-pass.txt"
& $restic backup "C:\Path\To\My\Data" "K:\Another\Data\Path"
```

<br>

Go to `File` => `Save As` and make sure the file is saved as `restic-backup.ps1`.  Then exit

<br>

Finally, we need to supply a `restic-pass.txt` which contains the repository password that was created in **Step 2**.  Supplying 
a password file is safer than writing the password directly into the powershell script as it will prevent the password from showing up
in any system logs.

Run the following powershell command:

```Powershell
E:\PowerShellScript> edit restic-pass.txt 
```

Then input your password into the editor without any trailing space

```Powershell
SomePasswordToMyRepositoryBackup
```

<br>

Go to `File` => `Save As` and make sure the file is saved as `restic-pass.txt`.  Then exit

<br>

---

## Step 9 — Setup Microsoft Task Scheduler to run the backup process

Now that all the scripts have been written, all that's left is to schedule the interval in which they are run.

1. In the windows `search` bar, type in `Task Scheduler`. and in the window next to the `Task Scheduler` icon make sure and select `run as administrator`
2. Inside the `Task Scheduler` app and on the right-hand side, select `Create Task`
3. Give the Task a name such as `Restic-Backup-Process` or something you can identify
4. Under Security options:
  - Make sure `When running the task, use the following account:` is set to the windows account that you will be logging into.  If this is your admin account then you can leave the current user.  If you will be logging into a different account, then you must `Change User or Group` to the appropriate account
  - Make sure `Run only when user is logged on` is selected
  - Make sure `Run with highest privileges` is selected
  - Make sure `Hidden` is selected
  - Make sure `Configure for Windows 10` is selected (as of this writing there is no Windows 11 option)

5. Select the `Triggers` tab at the top and select `New`
   - Make sure `Begin tte task` at the top is set to `On a schedule`
   - Make sure `Settings` is set to `Daily` and that `Recur every` is set to `1 days`
   - Under `Advanced settings` and `Repeat task every` is where you can set how often the scheduler fires off the backup script. I have mine set to every `5 minutes` but you can set your own intervals here.  Make sure `for a duration of:` is set to `1 day`.
   - Make sure at the botttom `Enabled` is checked
   - Click OK
     
6. Select the `Actions` tab at the top and select `New`
   - Make sure `Action` is set to `Start a program`
   - Under `settings` copy and paste the following code:

   ```Powershell
      wscript.exe
   ```
   
   <br>

   - Under `Add arguments (optional)` copy and paste the following code:
   
   ```Powershell
      "C:\Scripts\run-restic.vbs"
   ```
   
   <br>
  
   - Under `Start in (optional):` copy and paste the following code:
  
   ```Powershell
   C:\Scripts
   ```

  <br>

  - Finally, Click OK

7. Select the `Conditions` tab at the top
   - Make sure under `Power`:
     - `Start the task only if the computer is on AC power` is selected
     - `Stop if the computer siwthces to battery power` is selected
     - Click Ok
    
8. Finally, Click Ok on the window with all the tabs.


Congratulations! You now have a restic service which backups your selected drives/folders.  You can select your task by name in the main window of the Task Scheduler and in the right pane, select `Run`.
When the task is complete you should see under `Last Run Result` the message `The operation completed successfully. (0x0)`.  If you see any errors, check your log file located in `C:\Scripts\restic-run.log`







