# Teq. 19 The “save game” approach: Winning at 2048

> [!NOTE]
> **What this technique does:** The original book example runs the game **2048** in a container. Whenever you're doing well, you save the container's state. If you lose, you restore the save and carry on from there. This version uses `export`/`import` (as in Technique 11) to save a container's filesystem as a `.tar` backup and restore it as an image.
> To try it with the game, first run a container (e.g. a 2048 web image such as `docker run -d -p 8080:80 --name game alexwhen/docker-2048`), play a bit, then save it.

---

**Step 1: Export the Docker Container as a TAR file**

```bash
docker export Container_ID > filename.tar
```

> [!TIP]
> The "save game" step. It writes the container's whole current filesystem to a tar file. Get the ID from `docker ps`. Example:
> ```bash
> docker export game > game-save1.tar
> ```

**Step 2: Import the Tar File**

```bash
docker import - [Container_Name] < filename.tar
```

> [!TIP]
> Turns the backup tar into an **image** (the name you give here is the image name, despite the placeholder saying "Container_Name"). Example:
> ```bash
> docker import - my-backup < game-save1.tar
> ```

**Step 3: Verify the Docker Image**

```bash
docker images
```

> [!TIP]
> Your backup image (e.g. `my-backup:latest`) is listed. Every save you import becomes its own image, so you can keep several "save slots" (`my-backup:save1`, `my-backup:save2`, ...).

**Step 4: Pull an old image from repository**

```bash
docker pull filename-backup:latest
```

> [!TIP]
> This only works if the backup was previously **pushed to a registry** (Technique 17 or 9). A name like `filename-backup` with no username will be looked up on Docker Hub and fail with "repository does not exist". If the backup image is already local from Step 2, skip this step. If you pushed it, pull it as `username/my-backup:latest`.

**Step 5: Run the Restored Docker Container**

```bash
docker run -it my-backup:latest
```

> [!TIP]
> "Load game": starts a container from the saved state. Imported images have **no default command** (see Technique 11), so you will likely get `Error: No command specified`. Add one:
> ```bash
> docker run -it my-backup:latest /bin/sh
> ```
> For the 2048 web image, run its server command and map the port again, e.g. `docker run -d -p 8080:80 my-backup:latest nginx -g 'daemon off;'`.

---

> [!IMPORTANT]
> **Key takeaways**
> - Export a container and you have a save file. Import it and you have a restorable image.
> - Name the image in Step 2 consistently with the name you run in Step 5 (`my-backup` here).
> - `docker commit` (Technique 15) is the simpler alternative and keeps the image's metadata.
