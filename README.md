# Setting Up Restic For Windows 11 With Task Scheduler
This repository documents a **modern, minimal, and correct** way to create a __Time-Machine__ style backup service for Windows using Restic.
Actually, this service may be better than apple's own __Time-Machine__ and best of all it's free.  This guide will cover: 

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

