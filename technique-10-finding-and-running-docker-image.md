# Teq. 10 Finding and Running a Docker Image

> [!NOTE]
> **What this technique does:** Shows how to search Docker Hub from the terminal for an image, then download it.

---

**Step 1: Use the terminal to search the Docker Image**

```bash
docker search node
```

> [!TIP]
> Searches Docker Hub for images matching `node`. Columns:
> - `NAME`: the image name (`node` with no prefix = an official image; `user/node...` = community)
> - `DESCRIPTION`: short summary
> - `STARS`: popularity
> - `OFFICIAL`: `[OK]` means maintained by Docker / the vendor, so prefer these
>
> Filter example: `docker search --filter is-official=true node`

**Step 2: Run the Docker Image**

```bash
docker pull node
```

> [!TIP]
> `docker pull` **downloads** the image (tag `node:latest` by default). It doesn't run it. To actually run it:
> ```bash
> docker run -it node          # opens the Node.js REPL (type .exit to quit)
> docker run node node -v      # prints the Node version and exits
> ```
> `docker run` also pulls automatically if the image isn't downloaded yet. Pin a version for reproducibility, e.g. `node:20-alpine`.

---

> [!IMPORTANT]
> **Key takeaways**
> - `docker search` finds images, `docker pull` downloads them, and `docker run` starts a container (pulling first if needed).
> - Prefer official images and specific tags over `latest`.
