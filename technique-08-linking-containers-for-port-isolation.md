# Teq. 8 Linking Containers for Port Isolation

> [!NOTE]
> **What this technique does:** Sets up a classic two-tier app: a **MySQL** database container and an **Apache + PHP** web container. The `--link` flag lets the web container reach the database using the hostname `mysql`.
> `--link` is a **legacy** Docker feature. It still works, but user-defined networks (Technique 7) are the modern way to do this.

---

### To Link Containers for Port Isolation

**Step 1: Check the Networks - (Default: host, none and bridge)**

```bash
docker network ls
```

> [!TIP]
> `--link` only works on the default `bridge` network, which is where both containers will be placed.

**Step 2: Pull images of mysql and apache**

```bash
docker pull mysql/mysql-server:5.6
```

```bash
docker pull nimmis/apache-php5
```

> [!TIP]
> - `mysql/mysql-server:5.6`: MySQL 5.6 server image (the `:5.6` tag pins the version)
> - `nimmis/apache-php5`: a community image with Apache and PHP 5
>
> Both are old images. On Apple Silicon/ARM machines they may need `--platform linux/amd64`.

**Step 3: Run mysql and apache specifying a port number and**

```bash
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root123 -d mysql/mysql-server:5.6
```

```bash
docker run -tid -p 80:80 --name apache2 --link mysql nimmis/apache-php5
```

> [!TIP]
> **MySQL container:**
> - `-p 3306:3306`: publishes MySQL's port to the host
> - `-e MYSQL_ROOT_PASSWORD=root123`: `-e` sets an environment variable, and the image reads this to set the root password (a lab-only password)
> - `-d`: run in the background
>
> **Apache container:**
> - `-tid` = `-t` (terminal) + `-i` (interactive) + `-d` (detached)
> - `-p 80:80`: the web server at `http://localhost`
> - `--link mysql`: adds a `mysql` entry to the container's `/etc/hosts` and injects env variables like `MYSQL_PORT_3306_TCP_ADDR`, so PHP code can connect to host `mysql`
>
> Start MySQL **first**, because `--link` needs the target container to already exist. If port 80 or 3306 is already in use on your machine, change the host side (e.g. `-p 8080:80`).

**Step 4: Check the docker process state**

```bash
docker ps
```

> [!TIP]
> Both `mysql` and `apache2` should be `Up`. MySQL may show `(health: starting)` for a few seconds.

**Step 5: Check the networks of both containers**

```bash
docker inspect mysql
```

```bash
docker inspect apache2
```

> [!TIP]
> Scroll to `"NetworkSettings"`. Both containers are on `bridge` with IPs like `172.17.0.x`. In `apache2`'s output, look for `"Links": ["/mysql:/apache2/mysql"]`.

**Step 5: Execute apache in bash**

```bash
docker exec -ti apache2 bash
```

> [!TIP]
> Opens a bash shell **inside** the running `apache2` container. Try `cat /etc/hosts` and you will see the `mysql` entry that `--link` added.

**Step 6: Ping mysql to verify then**

> [!TIP]
> Inside the `apache2` shell, run `ping mysql`. Replies from `172.17.0.x` confirm the link works. If `ping` is missing, install it first with `apt-get update && apt-get install -y iputils-ping`. Type `exit` to leave the container shell.

---

> [!IMPORTANT]
> **Key takeaways**
> - `--link` connects one container to another by name on the default bridge network.
> - It is legacy. Prefer `docker network create` + `--network`.
> - For real isolation, don't publish the DB port (`-p 3306:3306`), so only the linked web container can reach MySQL.
