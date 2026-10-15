# Teq. 21 Rebuilding without the Cache

> [!NOTE]
> **What this technique does:** Docker caches every Dockerfile step as a layer. If a step and everything before it haven't changed, Docker **reuses** the cached layer instead of running it again, which is why rebuilds are fast.
> The problem: a step like `RUN apt-get update` *looks* identical to Docker, so it stays cached forever even though the package lists online have changed. `--no-cache` forces **every step to run fresh**.

---

**Step 1: Forcing a rebuild without using the cache**

```bash
docker build --no-cache .
```

> [!TIP]
> Run it in a folder with a Dockerfile (e.g. the one from Technique 20).
> - Normal rebuild: steps show `CACHED` and finish in about a second.
> - With `--no-cache`: every step runs again (downloads, installs), so it is slower but guaranteed fresh.
>
> As written, the result has no name (shows as `<none>` in `docker images`). Usually you add a tag:
> ```bash
> docker build --no-cache -t sample-image .
> ```

---

> [!IMPORTANT]
> **Key takeaways**
> - The cache speeds up builds but can give you stale results.
> - `--no-cache` ignores the cache for every step.
> - Use it when you need the latest packages or suspect a stale layer.
