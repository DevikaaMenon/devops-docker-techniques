# Teq. 6 Using Ports to Connect to Containers

> [!NOTE]
> **What this technique does:** Containers have their own isolated network, so a web server listening on port 80 *inside* a container can't be reached from your machine by default. **Port mapping** (`-p host:container`) forwards a port on your machine to a port inside the container.

---

### To Map Ports to Connect to Containers

**Step 1: Pull some image from Repository**

```bash
docker pull wordpress
```

> [!TIP]
> Downloads the official `wordpress` image (Apache + PHP + WordPress) from Docker Hub, without running it.

**Step 2: Map the port 10001 to port 80**

```bash
docker run -d -p 10001:80 --name container1 wordpress
```

> [!TIP]
> - `-d`: run in the background (detached)
> - `-p 10001:80`: **host port 10001 → container port 80** (the format is always `HOST:CONTAINER`)
> - `--name container1`: a readable name instead of a random one
>
> If you get "port is already allocated" or "name already in use", pick another port or remove the old container (`docker rm -f container1`).

**Step 3: Verify the container by navigating to http://localhost:10001 using a web browser**

> [!TIP]
> You should see the WordPress setup page (language selection). It will complain about the database later, because no MySQL is connected. That is expected, since the point here is only that the port mapping works. `docker ps` shows the mapping as `0.0.0.0:10001->80/tcp`.

---

> [!IMPORTANT]
> **Key takeaways**
> - Container ports are private until you publish them with `-p HOST:CONTAINER`.
> - You can run many containers that all use port 80 internally by mapping each one to a different host port.
