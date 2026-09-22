# Navidrome + Rclone Cloud Mount Stack for Portainer

This is a work in progress project so may not work as intended

This guide explains how to set up **Navidrome** (music server) connected to a mainstream cloud storage providers (such as Google Drive, OneDrive, or Dropbox) by mounting it locally using **Rclone** via Docker Compose inside **Portainer**.

## Links to Navidrome, Rclone and Portainer Repos:
 - https://github.com/navidrome/navidrome
 - https://github.com/rclone/rclone
 - https://github.com/portainer/portainer

---

## 🛠️ Prerequisites

1. **Docker & Portainer** up and running on your host system.
2. **Existing Rclone Configuration**: An `rclone.conf` file containing a pre-configured remote (e.g., Google Drive, S3, OneDrive, Mega).
3. **FUSE Support**: Ensure `/dev/fuse` is available on your Linux host.
4. **Host User ID/Group ID**: Determine your target non-root user's numeric UID and GID:
   ```bash
   id -u
   id -g
   ```

---

## 📁 Step 1: Prepare Host Directories

On your Docker host system, create the required directories and ensure correct permissions are assigned.

```bash
# 1. Directory to hold your rclone.conf
mkdir -p /path/to/rclone/config

# 2. Directory on the host where cloud storage will be mounted
mkdir -p /path/to/host/MusicDrive

# 3. Directory to store Navidrome's database & cache
mkdir -p /path/to/navidrome/data
```

Move your existing `rclone.conf` file into the rclone config directory:
```bash
cp /path/to/your/rclone.conf /path/to/rclone/config/rclone.conf
```

---

## ⚙️ Step 2: Configure Placeholder Values

Before pasting the Docker Compose stack into Portainer, download the `docker-compose.yml` file and alter it based on your own file system and preferences:

| Placeholder | Description | Example |
| :--- | :--- | :--- |
| `<YOUR_TIMEZONE>` | Time zone string | `Europe/London`, `America/New_York` |
| `<PATH_TO_RCLONE_CONFIG_DIR>` | Absolute host path containing `rclone.conf` | `/docker/rclone/config` |
| `<PATH_TO_HOST_MOUNT_DIR>` | Absolute host directory for FUSE mount | `/home/user/MusicDrive` |
| `<PATH_TO_NAVIDROME_DATA_DIR>` | Absolute host path for Navidrome DB | `/docker/navidrome/data` |
| `<RCLONE_REMOTE_NAME>` | Name of the remote in your `rclone.conf` | `gdrive`, `my_remote` |
| `<REMOTE_FOLDER_PATH>` | Folder path inside your cloud storage | `Music`, `Audio/Music` |
| `<YOUR_PUID>` | User ID on host system | `1000` |
| `<YOUR_PGID>` | Group ID on host system | `1000` |

---

## 🚀 Step 3: Deploy via Portainer

1. Log into your **Portainer** dashboard.
2. Select your environment (e.g., **Primary** or **local**).
3. Navigate to **Stacks** in the left sidebar menu.
4. Click **+ Add stack**.
5. Give your stack a name (e.g., `navidrome-rclone`).
6. Select **Web editor** and paste the contents of your updated `docker-compose.yml`
7. Click **Deploy the stack**.

---

## 🔍 Step 4: Verification & Initial Setup

1. **Check Logs**:
   - Go to **Containers** in Portainer.
   - Inspect the logs for `rclone-mount` to confirm the mount command succeeded without FUSE errors.
   - Inspect logs for `navidrome` to verify library indexing has started.
2. **Access Navidrome UI**:
   - Open a browser and navigate to `http://<YOUR_HOST_IP>:4533`.
   - On the first visit, create your **Admin account** username and password.

---

## ❓ Troubleshooting

* **Permission Denied / FUSE Errors**:
  - Verify `/dev/fuse` exists on the host.
  - Make sure `privileged: true`, `cap_add: [SYS_ADMIN]`, and `security_opt: [apparmor:unconfined]` remain in the `rclone` container definition.
* **Navidrome Shows Empty Library**:
  - Verify Rclone has mounted properly by inspecting contents on the host path (`ls /path/to/host/MusicDrive`).
  - Confirm mount propagation options are intact: `:shared` on Rclone and `:ro,rslave` on Navidrome.
  - Check user ownership (`<YOUR_PUID>:<YOUR_PGID>`) so Navidrome can read the mount point.
