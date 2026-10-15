# Teq 16. Docker Tagging

> [!NOTE]
> **What this technique does:** A **tag** is a human-readable name pointing to an image, in the form `repository:tag` (e.g. `ubuntu:22.04`, `myapp:v1`). `docker tag` adds **another name** to an existing image. It doesn't copy anything, so both names point to the same image ID. Tags are used for versioning (`v1`, `v2`, `latest`) and for preparing images to push to a registry (`username/repo:tag`).

---

**Step 1: List the images**

```bash
sudo docker images
```

> [!TIP]
> Note the `IMAGE ID` of the image you want to tag. Images from `docker commit` or `docker import` without a name show up as `<none>`, and tagging is how you name them.

**Step 2: Specify the name to the existing image**

```bash
sudo docker tag [IMAGE_ID] [new_image_name]
```

> [!TIP]
> Examples:
> ```bash
> sudo docker tag 3f2a1b9c8d7e ubuntu-nmap          # gets tag :latest by default
> sudo docker tag 3f2a1b9c8d7e ubuntu-nmap:v1       # explicit version tag
> sudo docker tag ubuntu-nmap:v1 myuser/ubuntu-nmap:v1   # by name instead of ID, ready for Docker Hub
> ```
> Run `sudo docker images` again and every tag shows the **same IMAGE ID**. Remove just a name with `docker rmi ubuntu-nmap:v1`. The image itself is only deleted when its last tag is removed.

---

> [!IMPORTANT]
> **Key takeaways**
> - A tag is a label, not a copy, so many tags can point to one image.
> - The format is `[registry/][username/]repository[:tag]`, and the tag defaults to `latest`.
> - Tag with a version (`v1`, `1.0.3`) so you can roll back. `latest` just means "untagged default", not "newest".
