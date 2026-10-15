# Teq. 18 Referring to a specific image in builds

Repeat the same process in mentioned in Writing a Dockerfile

---

> [!NOTE]
> **What this technique means:** The `FROM` line in a Dockerfile can refer to the base image in different ways. The more specific you are, the more **reproducible** your build is:
>
> | `FROM` line | What you get | Reproducible? |
> |---|---|---|
> | `FROM ubuntu` | Whatever `ubuntu:latest` is *today* | ❌ changes over time |
> | `FROM ubuntu:22.04` | Ubuntu 22.04 (still gets security updates) | ⚠️ mostly |
> | `FROM ubuntu@sha256:<digest>` | That exact image, byte for byte | ✅ always identical |
> | `FROM 3f2a1b9c8d7e` (local image ID) | A specific image on *your* machine only | ✅ locally only |

> [!TIP]
> **Applying it to the "Writing a Dockerfile" process** (e.g. the Redis Dockerfile from Technique 2):
> ```dockerfile
> # Instead of:  FROM alpine
> FROM alpine:3.19
>
> RUN apk add --update redis
> CMD ["redis-server"]
> ```
> Then build as usual:
> ```bash
> docker build -t secondfile .
> ```
> To find an image's digest: `docker images --digests` or `docker inspect --format='{{index .RepoDigests 0}}' alpine:3.19`

---

> [!IMPORTANT]
> **Key takeaways**
> - `latest` is a moving target, so the same Dockerfile can build differently next month.
> - Pin a version tag (`:3.19`) or, for full reproducibility, a digest (`@sha256:...`).
