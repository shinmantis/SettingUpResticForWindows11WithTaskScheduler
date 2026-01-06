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
>If you choose to backup and entire drive such as `H:\`, it's better to place all the contents of what you want to back up into a single folder; for example: `H:\MyData\AllOfYourDriveData`.
>The reason for this is, there are hidden directories in the root of any drive and if you attempt to back up the entire root of say `H:\` you may run into _inaccessible_ or _permission denied_
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

