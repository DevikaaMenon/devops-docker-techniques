# The Docker Daemon

## TECHNIQUE 1 - Open your Docker daemon to the world

> [!NOTE]
> **What this technique does:** Normally the Docker daemon (`dockerd`) only listens on a local Unix socket (`/var/run/docker.sock`), so only the same machine can talk to it. This technique makes the daemon *also* listen on TCP port **2375** on every network interface (`0.0.0.0`). After that, any device on the network can call the Docker REST API, e.g. `http://<your-ip>:2375/images/json` returns the list of images as JSON.

> [!WARNING]
> **Security:** Port 2375 is plain HTTP with **no authentication and no encryption**. Anyone who can reach that port has full root-level control of the machine through Docker. Only do this on a private lab network or hotspot, and undo it afterwards.

---

### For Ubuntu Users

**Step 1: Check the status of Docker**

```bash
sudo systemctl status docker
```

> [!TIP]
> Confirms Docker is installed and running (`active (running)`) before you change anything.

**Step 2: Open the systemd file of Docker**

```bash
sudo nano /lib/systemd/system/docker.service
```

> [!TIP]
> This is the systemd unit file that tells Linux *how* to start the Docker daemon. The `ExecStart=` line is the exact command used to start `dockerd`.

**Step 3: Comment the containerd.sock code under service**

```
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

> [!TIP]
> Put a `#` in front of this existing line so systemd ignores it. You are replacing the start command, not deleting it, so you can restore it later.

**Step 4: Add the code to access Docker Daemon**

```
ExecStart=/usr/bin/dockerd -H=fd:// -H=tcp://0.0.0.0:2375
```

> [!TIP]
> Each `-H` is a "host" the daemon listens on:
> - `fd://`: the normal local socket that systemd passes in (so local `docker` commands keep working)
> - `tcp://0.0.0.0:2375`: TCP port 2375 on **all** network interfaces (this is what opens it "to the world")

**Step 5: Reload the daemon & Restart Docker**

```bash
sudo systemctl daemon-reload ; systemctl restart docker
```

> [!TIP]
> `daemon-reload` makes systemd re-read the edited unit file, and `restart docker` starts Docker with the new `ExecStart`. Note that the second command has no `sudo`, so if it asks for authentication or fails, run it with `sudo`.

**Step 6: Check the IP address of the system**

```bash
sudo ifconfig
```

> [!TIP]
> Find your machine's IP (e.g. under `wlan0`/`wlp...` for Wi-Fi, look for `inet 192.168.x.x`). If `ifconfig` isn't found, install `net-tools` or use `ip a`.

**Step 7: Open the Web Browser and add port number - 2375 with /images/json**

> [!TIP]
> Example: `http://localhost:2375/images/json`. You should see raw JSON describing your Docker images. That is the Docker REST API answering over HTTP.

**Step 8: Connect another device using the same hotspot and
check the IP address of WiFi with port number - 2375/images/json**

> [!TIP]
> On a phone or laptop on the same network, open `http://<IP-from-step-6>:2375/images/json`. Seeing the same JSON proves the daemon is reachable remotely.

---

### For Non-Ubuntu Users

**Step 1: Move into Docker daemon configuration location**

```bash
cd /etc/docker
```

> [!TIP]
> `/etc/docker` holds Docker's configuration files, most importantly `daemon.json`.

**Step 2: Create a file daemon.json and add the code**

```bash
sudo nano daemon.json
```

```json
{
  "hosts": [
    "tcp://0.0.0.0:2375",
    "unix:///var/run/docker.sock"
  ]
}
```

> [!TIP]
> `daemon.json` is the other way to configure `dockerd`, using a config file instead of command-line flags. `hosts` lists everywhere the daemon should listen: TCP port 2375 on all interfaces, plus the normal local Unix socket.

**Step 3: Check the configuration**

```bash
cat daemon.json
```

> [!TIP]
> Prints the file so you can check the JSON is correct. A missing comma or bracket will stop Docker from starting.

**Step 4: Move out of the location**

```bash
cd
```

> [!TIP]
> `cd` with no argument returns to your home directory.

**Step 5: Change the System Administrator configuration**

```bash
cd /etc/systemd/system
```

> [!TIP]
> `/etc/systemd/system` is where admins put their own systemd overrides. Files here take priority over the package defaults in `/lib/systemd/system`.

**Step 6: Create a file override.conf  and the code**

```bash
sudo nano override.conf
```

```ini
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

> [!TIP]
> **Why this is needed:** the default service file starts `dockerd -H fd://`. If `hosts` is set in *both* the command line and `daemon.json`, Docker refuses to start because of the conflict.
> - The empty `ExecStart=` clears the original start command.
> - `ExecStart=/usr/bin/dockerd` starts it again with **no** `-H` flags, so the `hosts` in `daemon.json` are used.
>
> Note: systemd normally only picks up an override when it is placed at `/etc/systemd/system/docker.service.d/override.conf` (or created with `sudo systemctl edit docker`), followed by `sudo systemctl daemon-reload`. If Docker fails to start in Step 7, that is the likely reason.

**Step 7: Restart Docker**

```bash
sudo systemctl restart docker
```

> [!TIP]
> Restarts the daemon so it reads the new `daemon.json`. If it fails, check the reason with `sudo journalctl -u docker -n 50`.

**Step 8: Check the IP address of the system**

```bash
sudo ifconfig
```

> [!TIP]
> Same as Ubuntu Step 6: note your `inet` IP address.

**Step 9: Open the Web Browser and add port number - 2375 with /images/json**

> [!TIP]
> `http://localhost:2375/images/json` should return JSON.

**Step 10: Connect another device using the same hotspot and
check the IP address of WiFi with port number - 2375/images/json**

> [!TIP]
> `http://<your-IP>:2375/images/json` from another device proves remote access works.

---

> [!IMPORTANT]
> **Key takeaways**
> - The Docker CLI is just a client. It talks to the daemon over a REST API.
> - `-H` / `"hosts"` controls where the daemon listens.
> - Opening `tcp://0.0.0.0:2375` gives unauthenticated remote control. In real setups use TLS on port 2376 or an SSH tunnel instead.
