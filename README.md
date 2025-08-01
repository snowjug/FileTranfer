# Sharing Files from Windows to Ubuntu VM (CIFS/SMB)

## 🧰 Prerequisites

- Windows PC (Host or separate machine)

- Ubuntu VM (Guest, e.g., on VMware or VirtualBox)

- Both on same local network

- Ubuntu has package `cifs-utils` installed

## 🔧 Step 1: Enable File Sharing on Windows

1. Right-click on the folder (e.g., `Desktop` or any specific folder).

2. Click “Properties” → “Sharing” tab → “Advanced Sharing…”

3. Check ✅ “Share this folder”

4. Set the Share Name (e.g., `Desktop`)

5. Click Permissions, allow at least `Read` or `Read/Write` for desired users

6. Apply and OK

## 🔐 Step 2: Create/Enable a Windows Local or Samba-Compatible User

**Option A: Use a local Windows account**

- Username: `sambauser`

- Password: `root` (example)

- Ensure this user has access to the shared folder.

**Option B: Use existing account**

- Format: `MicrosoftAccount\you@example.com`

**To create local user manually:**

Settings → Accounts → Family & other users → Add someone else to this PC → Add user without Microsoft account

## 📦 Step 3: Install Required Package on Ubuntu

sudo apt update

sudo apt install cifs-utils smbclient

## 📂 Step 4: Discover Windows Shares from Ubuntu

Command: `smbclient -L //192.168.1.41 -U sambauser`

Replace `192.168.1.41` with the Windows machine IP

Enter password when prompted to see shared folders

## 📁 Step 5: Mount the Shared Folder

sudo mount -t cifs //192.168.1.41/Desktop /mnt/windows_desktop \

-o username=sambauser,password=root,uid=$(id -u),gid=$(id -g),vers=3.0

## 🔐 Step 6: Secure Mounting with Credential File

sudo mkdir -p /etc/smbcredentials

sudo nano /etc/smbcredentials/sambauser

Add the following lines:

username=sambauser

password=root

Then secure:

sudo chmod 600 /etc/smbcredentials/sambauser

## 🔁 Step 7: Auto-Mount on Boot with /etc/fstab

Edit fstab: `sudo nano /etc/fstab`

Add:

//192.168.1.41/Desktop /mnt/windows_desktop cifs credentials=/etc/smbcredentials/sambauser,uid=1000,gid=1000,vers=3.0,nounix,iocharset=utf8,file_mode=0755,dir_mode=0755 0 0

Reload systemd: `sudo systemctl daemon-reexec`

Mount all: `sudo mount -a`

## 🖥 Step 8: GUI Access (Optional)

- Open Files on Ubuntu

- Navigate to `/mnt/windows_desktop`

- Create bookmark for easy access

## ✅ Optional Enhancements

- Use systemd mount units for network-aware mounting

- Create desktop launchers for quick access

- Use rsync/unison for syncing files

## 🧪 Troubleshooting Tips

| Problem | Solution |

|---------|----------|

| NT_STATUS_LOGON_FAILURE | Check username/password format |

| mount error(13): Permission denied | Try vers=1.0, vers=2.0, or vers=3.0 |

| Cannot see shares | Allow SMB through Windows firewall |

| Auto-mount fails | Add x-systemd.automount or troubleshoot network timing |
