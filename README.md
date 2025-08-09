
# NFS Server & Client Simulation on Localhost

## Project Description

This repository documents a project to set up and configure a Network File System (NFS) server and client on a single Linux machine. The objective was to simulate a shared network environment using the loopback interface (`127.0.0.1`), demonstrating the core functionalities of NFS, including directory sharing, client mounting, and file transfer testing.

## Features

-   **NFS Server Installation:** Installation of the necessary packages on a Debian-based system (Kali Linux).
-   **Directory Sharing:** Creation and export of a shared directory using `/etc/exports`.
-   **Client Mounting:** Mounting the shared NFS directory to a local client-side path.
-   **File Sharing Test:** Successful creation and exchange of files between the server and client paths.
-   **Persistent Mount (Optional):** Configuration of the NFS mount to persist across reboots using `/etc/fstab`.
-   **Firewall Configuration (Optional):** Implementation of a basic firewall rule using `ufw` to allow NFS traffic from the loopback address.

## Prerequisites

-   A Linux machine (this guide uses a Debian-based distribution like Kali Linux).
-   `sudo` privileges to install packages and modify system files.

---

## Step-by-Step Guide

### Step 1: Install and Verify the NFS Server

First, update the package list and install the NFS kernel server.

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
````

Verify that the NFS service is active and running.

```bash
sudo systemctl status nfs-server
```

### Step 2: Configure and Export the Shared Directory

Create the directory that will be shared and configure the `/etc/exports` file.

```bash
sudo mkdir -p /srv/nfs_server
sudo chown nobody:nogroup /srv/nfs_server
sudo chmod 777 /srv/nfs_server
```

Edit `/etc/exports` to allow access from the local machine.

```bash
sudo nano /etc/exports
```

Add the following line to the file:

```
/srv/nfs_server 127.0.0.1(rw,sync,no_subtree_check)
```

Apply the changes and verify the export list.

```bash
sudo exportfs -a
showmount -e
```

### Step 3: Mount the Shared Directory on the Client Side

Create a mount point and mount the NFS share.

```bash
sudo mkdir -p /mnt/nfs_client
sudo mount -t nfs 127.0.0.1:/srv/nfs_server /mnt/nfs_client
```

Verify the mount with `mount | grep nfs`.

### Step 4: Test File Sharing

Create a file from the server side and confirm it is visible on the client side.

```bash
echo "This is a file created on the server." | sudo tee /srv/nfs_server/server_file.txt
ls -l /mnt/nfs_client
cat /mnt/nfs_client/server_file.txt
```

### Step 5: (Optional) Configure Persistent Mount

To ensure the NFS share is mounted on every boot, add an entry to `/etc/fstab`.

```bash
sudo nano /etc/fstab
```

Add the following line:

```
127.0.0.1:/srv/nfs_server /mnt/nfs_client nfs defaults 0 0
```

Reload the `systemd` daemon and test the mount configuration.

```bash
sudo systemctl daemon-reload
sudo mount -a
```

### Step 6: (Optional) Configure Firewall Rules

Install `ufw` and create a rule to allow NFS traffic from the loopback interface.

```bash
sudo apt-get install ufw
```

```bash
sudo ufw allow from 127.0.0.1 to any port nfs
sudo ufw enable
sudo ufw status verbose
```

-----

## Deliverables Summary

This section provides a quick overview of the final state of the project files and command outputs.

  - **Final `/etc/exports` file:**
    ```
    /srv/nfs_server 127.0.0.1(rw,sync,no_subtree_check)
    ```
  - **`showmount -e` output:**
    ```
    Export list for ziad:
    /srv/nfs_server 127.0.0.1
    ```
  - **`mount | grep nfs` output:**
    ```
    127.0.0.1:/srv/nfs_server on /mnt/nfs_client type nfs4 (rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=127.0.0.1,local_lock=none,addr=127.0.0.1)
    ```
  - **`ls -l /mnt/nfs_client` output:**
    ```
    -rw-r--r-- 1 root root 40 Aug  3 03:26 server_file.txt
    ```

