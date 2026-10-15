# Teq. 5 Using Docker in your Browser

Same Example as Docker Daemon discussed in

Teq. 1 - Open your Docker daemon to the World

> See [technique-01-open-docker-daemon.md](technique-01-open-docker-daemon.md)

---

> [!NOTE]
> **What this technique means:** Once the daemon listens on TCP port 2375 (Technique 1), its REST API is plain HTTP, so a **web browser can act as a Docker client** for read-only (`GET`) requests. URLs to try (replace `localhost` with your IP from another device):
>
> | URL | What it shows |
> |---|---|
> | `http://localhost:2375/images/json` | All images (same as `docker images`) |
> | `http://localhost:2375/containers/json` | Running containers (same as `docker ps`) |
> | `http://localhost:2375/containers/json?all=1` | All containers (same as `docker ps -a`) |
> | `http://localhost:2375/info` | System info (same as `docker info`) |
> | `http://localhost:2375/version` | Version (same as `docker version`) |

> [!WARNING]
> Same risk as Technique 1: the API is unauthenticated, so anything that can reach the port can also send `POST`/`DELETE` requests to create, run or delete containers.
