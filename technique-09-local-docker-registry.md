# Teq. 9  Setting Up a Local Docker Registry

> [!NOTE]
> **What this technique does:** A **registry** is a server that stores and serves Docker images. Docker Hub is the public one. Here you run your **own private registry** as a container on your machine, so you can push and pull images locally without the internet or a Docker Hub account.

---

**Step 1: To create and make the registry available**

```bash
docker run -d -p 5000:5000 -v $HOME/registry:/var/lib/registry registry:2
```

> [!TIP]
> - `registry:2`: the official registry image, version 2
> - `-d`: run in the background
> - `-p 5000:5000`: the registry's API is available at `localhost:5000`
> - `-v $HOME/registry:/var/lib/registry`: a **volume** that saves pushed images in `~/registry` on your machine, so they survive if the container is deleted

> [!TIP]
> **How to use it (try this after Step 1):**
> ```bash
> docker tag secondfile localhost:5000/secondfile   # name the image with the registry address
> docker push localhost:5000/secondfile             # upload to your registry
> curl http://localhost:5000/v2/_catalog            # list images in the registry -> {"repositories":["secondfile"]}
> docker pull localhost:5000/secondfile             # download it back
> ```
> The tag prefix (`localhost:5000/`) tells Docker which registry to push to. `localhost` is allowed over plain HTTP. Other hosts need TLS or an `insecure-registries` entry in `daemon.json`.

---

> [!IMPORTANT]
> **Key takeaways**
> - A registry is just a container (`registry:2`).
> - An image's name decides where it is pushed: `host:port/name`.
> - Use a volume so stored images aren't lost when the container goes away.
