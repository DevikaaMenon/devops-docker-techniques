# Teq 15. The “Save Game” Approach: Cheap Source Control

> [!NOTE]
> **What this technique does:** Like saving a video game, you can **snapshot a container's current state** at any point with `docker commit`. It turns the container (with everything you installed or changed inside it) into a new image. If you mess something up later, you start a fresh container from the saved image. It works as "cheap source control" for environments.

---

**Step 1: List all the images**

```bash
docker images
```

> [!TIP]
> See what images you already have before starting.

**Step 2: View the containers**

```bash
docker container ls
```

> [!TIP]
> Same as `docker ps`: lists running containers.

**Step 3: Run any image in the interactive mode**

```bash
docker run -it ubuntu:latest bin/bash
```

> [!TIP]
> - `-it`: interactive terminal, so you get a shell inside the container
> - `bin/bash`: the shell to run. It works because the container starts in `/`, though `/bin/bash` is the usual form.
>
> Your prompt changes to something like `root@3f2a1b9c8d7e:/#`. That hex string is the **container ID**, which you need in Step 7.

**Step 4: Update the Container inside the interactive mode**

```bash
apt-get update
```

> [!TIP]
> Runs **inside the container**. It downloads the latest package lists. The Ubuntu image ships with none, so `apt-get install` fails without this.

**Step 5: Install any tool in the container**

```bash
apt-get install nmap
```

> [!TIP]
> Installs `nmap` (a network scanning tool) into this container only, which changes its filesystem. Answer `Y` if prompted, or add `-y`.

**Step 6: Verify the installation process**

```bash
nmap --version
```

> [!TIP]
> Prints the nmap version, proving the change exists in this container. The base `ubuntu` image still does **not** have nmap.

**Step 7: Open a new terminal, commit the Container with ID and provide a new ame**

```bash
docker commit [CONTAINER_ID] [new_image_name]
```

> [!TIP]
> Run this in a **new terminal on the host** while the container is still running. Example:
> ```bash
> docker commit 3f2a1b9c8d7e ubuntu-nmap
> ```
> This is the **"save game"**: the container's current filesystem becomes a new image layer on top of `ubuntu`. Find the ID with `docker ps` or from the container's prompt.

**Step 8: Exit the container**

```bash
exit
```

> [!TIP]
> Back in the first terminal, this leaves the shell. Bash was the container's main process, so the container **stops**.

**Step 9: Display the list of launched containers**

```bash
docker ps
```

> [!TIP]
> The Ubuntu container no longer appears because it has stopped. `docker ps -a` shows it with status `Exited`.

**Step 10: Verify the image with**

```bash
docker images
```

> [!TIP]
> Your new image (e.g. `ubuntu-nmap`) is listed. It's larger than `ubuntu` because it includes nmap. To "load the save":
> ```bash
> docker run -it ubuntu-nmap bash
> nmap --version   # already installed
> ```

---

> [!IMPORTANT]
> **Key takeaways**
> - `docker commit` turns a container's current state into a reusable image.
> - It's quick for experiments and checkpoints, but it isn't reproducible: nobody can see *how* the image was made. For real projects, write the steps in a Dockerfile.
