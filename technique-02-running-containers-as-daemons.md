# TECHNIQUE 2 - Running containers as daemons

> [!NOTE]
> **What this technique does:** You build a small image that runs a Redis server, then run it two ways:
> - **Foreground (attached):** the container's output takes over your terminal until you stop it.
> - **Detached (`-d`):** the container runs in the background like a service ("daemon") and you get your prompt back immediately.

---

**Step 1: Create a new folder - secondfile**

```bash
mkdir secondfile
```

> [!TIP]
> Each image gets its own folder because `docker build` sends the whole folder (the "build context") to the daemon.

**Step 2: Move into the folder**

```bash
cd secondfile
```

**Step 3: Create a Dockerfile and add the code**

```bash
sudo nano Dockerfile
```

```dockerfile
# To run a simple container

# Use an existing docker image as a base
FROM alpine

# Download and install a dependency
RUN apk add --update redis

# Communicate the process
CMD ["redis-server"]
```

> [!TIP]
> - `FROM alpine`: starts from Alpine Linux, a very small (~5 MB) base image.
> - `RUN apk add --update redis`: runs at **build time**. `apk` is Alpine's package manager, and this installs Redis into the image.
> - `CMD ["redis-server"]`: the default command that runs at **container start time**.
>
> The file must be named exactly `Dockerfile` (capital D, no extension).

**Step 4: Build the Dockerfile**

```bash
sudo docker build -t secondfile .
```

> [!TIP]
> - `-t secondfile` tags (names) the image `secondfile` (`:latest` is implied).
> - `.` means use the current folder as the build context.

**Step 5: Check the Images**

```bash
sudo docker images
```

> [!TIP]
> `secondfile` should now appear in the list, along with the `alpine` base image it was built from.

**Step 6: Run the Image - secondfile**

```bash
docker run secondfile
```

> [!TIP]
> Runs in the **foreground**. You will see the Redis startup log ending with *"Ready to accept connections"*. (This line has no `sudo`; if you get a permission error on `docker.sock`, add `sudo`.)

**Step 7: Observe the prompt being witheld by the process & close the connection - ctrl + c**

> [!TIP]
> Your terminal is stuck because it is attached to the container's main process. `Ctrl + C` sends SIGINT, Redis shuts down, and the container stops.

**Step 8: Re-run the Image in the detached mode**

```bash
sudo docker run -d secondfile
```

> [!TIP]
> `-d` = **detached**. Docker starts the container in the background and prints only the long container ID.

**Step 9: Observe the prompt exited**

> [!TIP]
> You get your shell prompt back straight away, but Redis is still running in the background.

**Step 10: Check the process state of docker**

```bash
sudo docker ps
```

> [!TIP]
> Lists **running** containers. You should see `secondfile` with status `Up ...` and port `6379/tcp` (the Redis port). Useful follow-ups:
> - `sudo docker logs <id>`: see its output
> - `sudo docker stop <id>`: stop it
> - `sudo docker ps -a`: also show stopped containers (like the one from Step 6)

---

> [!IMPORTANT]
> **Key takeaways**
> - `RUN` runs at build time and `CMD` runs at container start.
> - With no flag the container is attached to your terminal. With `-d` it runs in the background, which is how you run long-lived services.
