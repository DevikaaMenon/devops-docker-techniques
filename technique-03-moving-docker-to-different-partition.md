# TECHNIQUE 3 - Moving Docker to a different partition

> [!NOTE]
> **What this technique does:** Docker stores all images, containers and volumes in its **data root**, which is `/var/lib/docker` by default. If that disk or partition fills up, you can point Docker at a different location with `--data-root`.

---

**Step 1: Stop the Docker service**

```bash
sudo systemctl stop docker.service
```

> [!TIP]
> Stop the daemon first, because two daemons cannot run at the same time.

**Step 2: Move the contents of Docker to a different location**

```bash
sudo dockerd --data-root /home/deepika/mydocker
```

> [!TIP]
> This starts the Docker daemon **manually in the foreground** with `/home/deepika/mydocker` as its data root. Docker creates its folder structure there (`image/`, `containers/`, `volumes/`, `overlay2/`, ...).
> - Replace `deepika` with your own username/path.
> - The terminal stays busy showing daemon logs. Open a **new terminal** for the next steps, and press `Ctrl + C` here to stop it.
> - Despite the step title, this does **not** copy your existing images. It starts with an empty data root. To keep old images, copy `/var/lib/docker` there first (e.g. `sudo rsync -aP /var/lib/docker/ /home/deepika/mydocker/`).
> - To make it permanent, put `{ "data-root": "/home/deepika/mydocker" }` in `/etc/docker/daemon.json`.
> - If you get `docker.socket` errors, also stop it with `sudo systemctl stop docker.socket`.

**Step 3: Start the Docker service**

```bash
sudo systemctl start docker.service
```

> [!TIP]
> Starts the normal systemd-managed Docker again. Unless you set `data-root` in `daemon.json`, it goes back to `/var/lib/docker`. Stop the manual `dockerd` from Step 2 (`Ctrl + C`) before running this.

**Step 1: Check the folder creation and contents by navigating into the directory**

```bash
cd /home/deepika/mydocker
```

> [!TIP]
> (This is the final step. The original numbers it "Step 1" again.) Run `sudo ls` inside, because the folder is owned by root. You should see Docker's directory layout, which proves `--data-root` worked.

---

> [!IMPORTANT]
> **Key takeaways**
> - Everything Docker stores lives in one directory, the data root.
> - `--data-root` (flag) or `"data-root"` (in `daemon.json`) moves it.
> - Moving it does not migrate existing data unless you copy it yourself.
