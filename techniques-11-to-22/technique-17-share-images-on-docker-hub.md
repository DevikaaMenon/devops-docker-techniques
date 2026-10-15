# Teq. 17 To share images on the Docker Hub

> [!NOTE]
> **What this technique does:** Docker Hub (hub.docker.com) is the public registry. Once you **push** an image there, anyone (or just you, if it's private) can `docker pull` it on any machine. The flow is **login → tag with your username → push**.

---

**Step 1: Sign-up & Sign-in in Docker Hub repository**

```bash
docker login
```

Enter the username and password credentials of Docker Hub

> [!TIP]
> Create a free account at hub.docker.com first. Newer Docker versions may show a **device code and URL** to confirm in the browser instead of asking for a password. You can also use a Personal Access Token (Account Settings → Security) as the password. Success shows `Login Succeeded`.

**Step 2: List all the images in the local**

```bash
docker images
```

> [!TIP]
> Check what you have locally that could be pushed.

**Step 3: Pull an image if none is available**

```bash
docker pull ubuntu:latest
```

> [!TIP]
> Downloads the Ubuntu image to use as a starting point.

**Step 4: Make necessary changes or modifications**

```bash
apt-get update
```

```bash
apt-get install nmap
```

> [!TIP]
> These run **inside a container**, not on the host. The full flow is the same as Technique 15:
> ```bash
> docker run -it ubuntu:latest bash      # start a container
> apt-get update && apt-get install -y nmap   # (inside) make changes
> docker commit <CONTAINER_ID> local-image:tagname   # (new host terminal) save as an image
> ```

**Step 5: Tag the Docker image**

```bash
docker tag local-image:tagname username/repository:tagname
```

> [!TIP]
> Docker Hub only accepts images named **`<your-username>/<repo>`**. Example:
> ```bash
> docker tag ubuntu-nmap:latest devikamenon/ubuntu-nmap:v1
> ```
> Replace `username` with your **exact Docker Hub username**, otherwise the push fails with `denied: requested access to the resource is denied`.

**Step 6: Push the repository to Docker Hub**

```bash
docker push username/repository:tagname
```

> [!TIP]
> Uploads the image layers. Layers that already exist on Hub (like the base `ubuntu` layers) are skipped ("Mounted from library/ubuntu"). Afterwards:
> - The repo appears at `https://hub.docker.com/r/<username>/<repository>`
> - Anyone can get it with `docker pull username/repository:tagname`

---

> [!IMPORTANT]
> **Key takeaways**
> - `docker login` → `docker tag` (as `username/repo:tag`) → `docker push`.
> - The image name decides where it is pushed. With no registry host in the name, it goes to Docker Hub.
> - Public repos are visible to everyone, so never bake passwords or keys into images you push.
