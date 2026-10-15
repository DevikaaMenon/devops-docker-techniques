# Teq. 4 Using socat to Monitor Docker API Traffic

> [!NOTE]
> **What this technique does:** `socat` is a relay that connects two endpoints and forwards data between them. Here it creates a **new** socket `/tmp/dockerapi.sock` that forwards everything to the real `/var/run/docker.sock`. With `-v`, it prints every request and response it relays, so you can see the raw HTTP the Docker client and daemon exchange.
>
> Install it first if needed: `sudo apt install socat`

---

### To Monitor traffic

**Step 1: Run socat using**

```bash
sudo socat -v UNIX-LISTEN:/tmp/dockerapi.sock,fork UNIX-CONNECT:/var/run/docker.sock
```

> [!TIP]
> - `-v`: verbose, prints the traffic passing through
> - `UNIX-LISTEN:/tmp/dockerapi.sock`: create and listen on a new Unix socket
> - `fork`: handle each new connection in a child process, so it keeps working after the first request
> - `UNIX-CONNECT:/var/run/docker.sock`: forward everything to the real Docker socket
>
> This terminal stays busy, which is expected. The traffic appears here.

**Step 2: To see the request and response [in a new terminal]**

```bash
curl --unix-socket /var/run/docker.sock http://localhost/images/json
```

> [!TIP]
> This calls the Docker API directly with `curl` and returns your images as JSON.
> Note that this command uses the **real** socket (`/var/run/docker.sock`), so the request skips socat and **nothing will appear** in the socat terminal. To watch the traffic, send the request through the proxy socket instead:
> - `sudo curl --unix-socket /tmp/dockerapi.sock http://localhost/images/json`
> - or `sudo docker -H unix:///tmp/dockerapi.sock images`
>
> In the socat window, `>` lines are requests (e.g. `GET /images/json HTTP/1.1`) and `<` lines are responses.

**Step 3: Use the following command to remove the socket [if it exists]**

```bash
sudo rm -f /tmp/dockerapi.sock
```

> [!TIP]
> If socat stopped uncleanly, the socket file may still exist and Step 1 will fail with "Address already in use". Removing it fixes that. `-f` stops it complaining if the file is missing.

**Step 4: Kill the socket**

```bash
sudo pkill socket
```

> [!TIP]
> `pkill` kills processes **by name**. The process is called `socat`, not `socket`, so this command will not match it. Use `sudo pkill socat`, or just press `Ctrl + C` in the socat terminal.

---

> [!IMPORTANT]
> **Key takeaways**
> - Every `docker` command is just an HTTP request to the daemon's REST API over a Unix socket.
> - `socat` lets you sit in the middle of that connection and see the exact API calls, which helps with debugging and learning the API.
