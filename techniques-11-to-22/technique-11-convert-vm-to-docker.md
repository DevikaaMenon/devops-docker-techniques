# Teq. 11 To convert a VM to Docker

> [!NOTE]
> **What this technique does:** The idea is "take a whole running machine and turn it into an image". A real VM conversion means tarring the VM's filesystem and importing it. This lab simulates that with a container:
> 1. `docker export` dumps a container's **entire filesystem** into a `.tar` file (like taking a snapshot of a VM's disk).
> 2. `docker import` turns that `.tar` back into a **new image**.
>
> The result is a single-layer image with no history and no metadata (no `CMD`, no `EXPOSE`, no `ENV`), which is why you have to tell it what command to run in Step 7.

---

**Step 1: Create Nginx Container**

```bash
docker run -d -p 80:80 --name nginx nginx
```

> [!TIP]
> Starts the official `nginx` web server in the background (`-d`), maps host port 80 to container port 80, and names the container `nginx`. This plays the role of the "machine" being converted. Check it at `http://localhost`.

**Step 2: Displaying Running Container**

```bash
docker ps
```

> [!TIP]
> Confirms the `nginx` container is `Up`. Note its name or ID for the next step.

**Step 3: Export the Container with the Container name or ID**

```bash
docker export  nginx > mynginx.tar
```

> [!TIP]
> `docker export` writes the container's full filesystem as a tar stream, and `>` saves it to `mynginx.tar` in the current folder. It captures **files only**, not image layers, history, env variables or the start command.

**Step 4: Import the created tar file as a new Image**

```bash
docker import - mynginx < mynginx.tar
```

> [!TIP]
> - `-` means read the tar from standard input
> - `< mynginx.tar` feeds the file in as that input
> - `mynginx` is the name of the new image (tag `latest`)
>
> Equivalent shorter form: `docker import mynginx.tar mynginx`

**Step 5: List all the images**

```bash
docker images
```

> [!TIP]
> `mynginx` now appears. Compare it with `nginx`: it is a **flattened** single-layer image. `docker history mynginx` shows only one entry ("Imported from -").

**Step 6: List all the files in the current location**

```bash
ls -l
```

> [!TIP]
> Shows `mynginx.tar` on your host, typically around 180–200 MB. That is the "VM disk" you exported.

**Step 7: Load Docker image into your system:**

```bash
docker run --name container_nginx -d mynginx:latest nginx -g 'daemon off;'
```

> [!TIP]
> Despite the title, this **runs** a new container from the imported image.
> - The imported image lost its default `CMD`, so you must give the command yourself: `nginx -g 'daemon off;'`
> - `daemon off;` keeps nginx in the foreground. Without it nginx backgrounds itself, the main process exits, and the container stops immediately.
> - No `-p` is given, so it isn't reachable from the browser (port 80 is already used by the first container anyway). Add e.g. `-p 8080:80` to test it.

**Step 8: Displaying Running Container**

```bash
docker ps
```

> [!TIP]
> Both `nginx` (original) and `container_nginx` (from the imported image) should be `Up`.

---

> [!IMPORTANT]
> **Key takeaways**
> - `export`/`import` work on a container's **filesystem**, which is how you bring an existing machine into Docker.
> - Imported images are flattened and lose metadata (`CMD`, `ENV`, ports), so you supply those yourself.
> - Don't confuse this with `save`/`load`, which work on **images** and keep layers and metadata.
