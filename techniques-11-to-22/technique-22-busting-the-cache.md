# Teq. 22 Busting the Cache

Comparision of with and without Cache

---

> [!NOTE]
> **What this technique means:** `--no-cache` (Technique 21) throws away the cache for **all** steps. **Cache busting** is the more surgical approach. You change something in the Dockerfile at a chosen point, which invalidates the cache **from that step onwards**, while steps above it stay cached and fast.
>
> **Rule:** once one step's cache is invalidated, **every step after it** is rebuilt too.

> [!TIP]
> **Comparison: with vs without cache** (using the Dockerfile from Technique 20)
>
> ```bash
> docker build -t sample-image .            # 1st build: every step runs
> docker build -t sample-image .            # 2nd build: every step shows CACHED, very fast
> docker build --no-cache -t sample-image . # every step runs again, slow
> ```
>
> | | With cache | Without cache (`--no-cache`) |
> |---|---|---|
> | Speed | Seconds | Full time (downloads, installs) |
> | Output | `CACHED` on unchanged steps | Every step runs |
> | Freshness | May reuse stale `apt-get update` results | Always fresh |
> | When to use | Normal day-to-day builds | Need latest packages / debugging stale layers |

> [!TIP]
> **Ways to bust the cache at a specific step:**
> 1. **Edit a file that's `ADD`/`COPY`'d.** Change `sample.txt` and rebuild. Only `ADD sample.txt .` and later steps rerun, and `RUN apt-get -y update` stays `CACHED`.
> 2. **Change the instruction text.** Even adding a comment-like change to the command (e.g. `RUN echo bust1 && apt-get -y update`) makes Docker treat it as a new step.
> 3. **Use a build argument:**
>    ```dockerfile
>    FROM ubuntu:latest
>    ARG CACHEBUST=1
>    RUN apt-get -y update
>    ADD sample.txt .
>    ```
>    ```bash
>    docker build --build-arg CACHEBUST=$(date +%s) -t sample-image .
>    ```
>    A new value each time invalidates the cache from where `CACHEBUST` is first used, so the `RUN` and everything below it is rebuilt.

---

> [!IMPORTANT]
> **Key takeaways**
> - The cache is invalidated at the first changed step, and everything after it rebuilds.
> - Cache busting rebuilds only from a chosen point, while `--no-cache` rebuilds everything.
> - Put steps that rarely change (installing packages) **above** steps that change often (copying your code) to get the most out of the cache.
