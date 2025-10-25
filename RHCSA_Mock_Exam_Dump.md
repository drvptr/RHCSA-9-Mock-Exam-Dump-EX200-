# RHCSA Mock Exam Dump (EX200)

## General Information
- You are provided with **two systems**:
- `servera.example.com` — main host
- `serverb.example.com` — secondary host
- Root password for **servera** system: `redhat`
- Unless specified, **all commands should persist after reboot**. All configurations must survive system restart.
- SELinux and the firewall must remain enabled and properly configured.
- Total exam time: 3 hours

---

## 1. Configure Networking and Host Resolution
Set static IPs:
 - `servera`: `10.0.0.10`
 - `serverb`: `10.0.0.11`
 - Netmask: `255.255.255.0`
 - Gateway: `10.0.0.1`
 - DNS: `1.1.1.1`, `8.8.8.8`
Add the following lines to `/etc/hosts` on both systems:
 ```
 10.0.0.10 servera.example.com servera
 10.0.0.11 serverb.example.com serverb
 ```

---

## 2. Configure a Local Repository via FTP
On **serverb**, mount the RHEL ISO file `rhel9.iso` on `/mnt/iso` using loopback.
Install and enable the **vsftpd** service.
Share `/mnt/iso` contents through FTP, anonymously.
Ensure the FTP service starts automatically on boot.
On **servera**, configure a local repository:
Bases URLs: `ftp://serverb.example.com/BaseOS`, `ftp://serverb.example.com/AppStream`
Verify with `dnf repolist`.

---

## 3. Manage User Accounts
Create the following users and groups:
 - Group `admin`
 - User `alex`, UID `1050`, primary group `admin`
 - User `nancy`, primary group `admin` and make that when the user `nancy` logs in, the message `"Welcome Nancy!"` is displayed.
 - User `chopin`, and make it so that all files and directories that this user creates are read-only (`r--r--r--` for files and `dr-xr-xr-x` for directories)
 - User `bob`, secondary group `admin`
 - User `guest`, with no interactive shell
Set password `12345` for `alex` and `bob`.
Configure `/etc/login.defs` so that new users meet the following requirements:
 - Maximum password age = **30 days**
 - Accounts become inactive after **90 days** 
 - Password history remembers **3** previous passwords.

---

## 4. Configure Sudo Access
Allow members of group `admin` to run **only** `dnf` and `echo` commands without a password.

---

## 5. Manage Disk and LVM
Create a new partition on`/dev/vda`
Create a **VG** `datastore` with PE size **8MiB**.
Create a **LV** `data_lv` of **500MiB**.
Format with XFS and mount at `/data`.
Extend the LV to **1GiB** and grow the filesystem.
Create a **LV** `image` of 20 extents. 
Format with ext4 and mount at `/image`.
Add the label `image1` to the `image` file system.
Create a **swap** partition of **512MiB** and enable it persistently.

---

## 6. Create a Thin Pool and Snapshot
Create a **thin pool** named `tpool` of size **1GiB** inside `datastore`.
Enable autoextend threshold at 80%.
Create a thin LV `thinvol` (2GiB virtual size) using this pool.
Create a snapshot of `thinvol` named `thinvol_snap`.
Restore from the snapshot after deleting a test file.

---

## 7. Manage Scheduled Jobs
 Create a cron job for user `alex` that:
 - Runs every 5 minutes daily between **18:00–20:00**
 - It should write all lines containing the word "error" from `/var/log/messages` to `/home/alex/err.log`.

---

## 8. Configure SSH Service
On **serverb**, configure SSH:
 - Allow root login **with key only**.
 - Deny SSH access to all other users.
 - Set MaxAuthTries to **3**.
Copy root’s public key from `servera` to `serverb` to allow passwordless access.

---

## 9. Manage NFS and Autofs
On **serverb**, create users from `remoter0` to `remoter99`, with home directory `/rhome/remoterX` (X - where X is the number)
Share `/rhome` via NFS:
 - Export to `servera.example.com` only
 - Allow read-write
On **servera**, configure `autofs` so every `remoterX` automatically mounts from `serverb:/rhome` to the `/rhome`.
Ensure that each user’s home directory is correctly mounted at `/rhome/remoterX` when logging in on **servera**, and that the same directory exists on **serverb**.

---

## 10. Configure HTTPD on a Non-standard Port
On **serverb**, install **httpd** service.
Configure the listen port to **88**, and serve files from `/containers` (create it).
Create `/containers/index.html` with the following content:
 ```
 FROM docker.io/library/mysql-server
 CMD ["mysqld"]
 ```
Make sure the server is accessible from **servera**.

---

## 11. Configure a Containerized MySQL
On **servera**, download the file from `http://serverb.example.com:88/`.
Build the image with the name `mysqld` and version `1.0`.
Create the container named `dbserver`:
 - Port mapping: `3307:3306`
 - Container volume `/var/lib/mysql` must be mounted to the `/home/alex/container-mysql-lib/`
 - Root password: `redhat`
Generate a systemd unit with name `container-dbserver`, enable it for autostart by an `alex` user.

---

## 12. Log Management Script
Create a script `/usr/local/bin/logrotate.sh`:
 - Find all `.log` files older than 7 days under `/var/log`
 - Copy them to `/archive`
 - Log all errors and actions.
The script should be owned by and executable only by user `alex`.

---

## 13: Configure a Shared Directory for a Project Team
On **servera** create a collaborative directory `/projects` with following requirements
Group ownership is `admin`. 
The directory should be readable, writable, and accessible to members of `admin`, but not to any other users (except for root).
Files created in `/projects` automatically have group ownership set to the `admin` group. Also ensure only file owners can delete their own files `/projects`. Despite this, the user `nancy` can delete all files, including those that do not belong to her.

---

## 14. Configure Time Synchronization Using Chrony
On serverb.example.com:
Configure Chrony as a server with stratum 8. Make sure it accepts NTP requests from other hosts on the network.
On servera.example.com:
Disable any default external NTP sources. Configure chronyd to synchronize time from serverb.example.com.

---

## 15. Configure tuned
Check your system's default tuning profile and set it as per recommended for this system.

---

## 16. Create a Compressed Archive of System Files
Create a compressed archive of the /etc, /home, and /var/log directories.
The archive should be named /root/system_backup-<date>.tar.gz, where <date> is the current date in YYYY-MM-DD format. Use the tar utility with compression (gzip).

---

## 17. Change the Default Systemd Target
Identify the current default systemd target.
Change the system’s default boot target to multi-user.target.
Reboot and verify that the system starts in non-graphical mode.

---

## 18. Update Kernel and Modify GRUB Timeout
Update the kernel, using DNF. Configure GRUB to have a boot menu timeout of 3 seconds.
Ensure that after reboot, the system boots into the latest kernel by default.

---

## 19. Configure Stratis Storage
Install the required packages for Stratis.
Create a Stratis pool named pool1 using the device /dev/vdb.
Create a filesystem named stratisfs1 in the pool1 pool.
Mount the filesystem persistently on /stratisdata using its UUID.
Ensure that after reboot, the Stratis pool and filesystem are available automatically.

---

## 20. Root password reset
Set the root password as redhat on **serverb** (without having the root password)
