# Teq. 7 Allowing Container Communication

> [!NOTE]
> **What this technique does:** Containers on Docker's default `bridge` network can't find each other **by name**. If you create a **user-defined network** and attach containers to it, Docker provides built-in DNS, so `container1` can reach `container2` just by using its name.

---

### To Allow Container Communication & To Link Containers for Port Isolation

**Step 1: Check the Networks - (Default: host, none and bridge)**

```bash
docker network ls
```

> [!TIP]
> The three built-in networks:
> - `bridge`: the default for new containers (private internal network, no name resolution)
> - `host`: the container shares the host's network directly
> - `none`: no networking

**Step 2: Create a user-defined network:**

```bash
docker network create my_network
```

> [!TIP]
> Creates a new bridge-type network. Unlike the default `bridge`, it has **automatic DNS between containers**.

**Step 3: Create two containers (in seperate shells)**

```bash
docker run --name container1 -it firstfile:latest
```

```bash
docker run --name container2 -it firstfile:latest
```

> [!TIP]
> - `-it`: interactive terminal, which drops you into a shell inside the container
> - `firstfile:latest` is an image you built earlier (an Ubuntu/Debian-based one, since Step 5 uses `apt-get`). If you don't have it, use `ubuntu` instead.
> - Run each command in its own terminal and keep both open.

**Step 3: Connect two containers**

```bash
docker network connect my_network container1
```

```bash
docker network connect my_network container2
```

> [!TIP]
> Run these in a **third** terminal on the host, not inside the containers. This attaches the already-running containers to `my_network`, so each container is now on both `bridge` and `my_network`.
> (Shortcut for next time: `docker run --network my_network ...` attaches at start.)

**Step 4: Inspect the Network - my_network (find the connected containers inside the Containers)**

```bash
docker network inspect my_network
```

> [!TIP]
> Look at the `"Containers"` section of the JSON. Both `container1` and `container2` should be listed with their IP addresses on this network (e.g. `172.18.0.2`, `172.18.0.3`).

**Step 5: Install iputils-ping inside both the containers**

```bash
apt-get install iputils-ping
```

> [!TIP]
> Run this **inside each container's shell** (the terminals from Step 3). Minimal images don't include `ping`. If you get "Unable to locate package", run `apt-get update` first.

**Step 6: Ping the containers using**

```bash
docker exec -it container1 ping container2
```

> [!TIP]
> Run this from the host. `docker exec` runs a command inside an already-running container. Here `container1` pings `container2` **by name**, and replies like `64 bytes from container2.my_network (172.18.0.3)` show that Docker's DNS resolved the name. Press `Ctrl + C` to stop.

---

> [!IMPORTANT]
> **Key takeaways**
> - Use user-defined networks for container-to-container communication.
> - They give you DNS by container name, so you don't need to hard-code IPs.
> - Only containers on the same network can talk to each other, which keeps other containers isolated.
