# Teq. 20  Injecting Files into the Image using ADD

> [!NOTE]
> **What this technique does:** The `ADD` instruction copies files from your machine (the build context) **into the image at build time**, so every container made from it already has the file.
> `ADD` has two extras over `COPY`: it **auto-extracts** local `.tar`/`.tar.gz` archives and can download from URLs. That's why the last step is called "Verifying the Extraction". For a plain file like `sample.txt`, `ADD` and `COPY` behave the same.

---

**Step 1: Create a Text file with some contents and save it inside the Dockerfile folder**

> [!TIP]
> For example:
> ```bash
> mkdir addfile && cd addfile
> echo "Hello from the host machine" > sample.txt
> ```
> The file **must** be inside the same folder as the Dockerfile (the build context). Docker can't `ADD` files from outside it, such as `../file.txt`.

**Step 2: Creating the Dockerfile**

```dockerfile
# Use the latest Ubuntu image as the base
FROM ubuntu:latest

# Update the package list
RUN apt-get -y update

# Add the text file from the specified path on the host to the current directory in the container
ADD sample.txt .
```

> [!TIP]
> - `FROM ubuntu:latest`: the base image
> - `RUN apt-get -y update`: refreshes package lists (not strictly needed here, but it adds a layer you will see cached in Technique 21/22)
> - `ADD sample.txt .`: copies `sample.txt` into the image's **current working directory**. With no `WORKDIR` set, that's `/`, so the file ends up at `/sample.txt`.
>
> Save it as `Dockerfile` (e.g. `nano Dockerfile`).

**Step 3:  Building the Docker Image**

```bash
docker build -t sample-image .
```

> [!TIP]
> Builds the image and names it `sample-image`. In the output, the `ADD sample.txt .` step shows the file being sent in.

**Step 4: List the images**

```bash
docker images
```

> [!TIP]
> `sample-image` should be listed.

**Step 5: Running the Docker Container**

```bash
docker run -it sample-image bash
```

> [!TIP]
> Starts a container and opens a bash shell inside it.

**Step 6: Verifying the Extraction**

```bash
ls
```

> [!TIP]
> Run inside the container, at `/`. `sample.txt` appears alongside `bin`, `etc`, `usr`, etc. Check its content with `cat sample.txt`.
> **To see actual extraction:** put a `.tar` file in the folder (`tar -cf files.tar sample.txt`) and use `ADD files.tar /data/`. The **contents** are unpacked into `/data/`, not the tar itself.

---

> [!IMPORTANT]
> **Key takeaways**
> - `ADD <src> <dest>` bakes host files into the image at build time.
> - It auto-extracts local tar archives, while `COPY` never does.
> - Best practice: use `COPY` for plain files and `ADD` only when you want the extraction.
