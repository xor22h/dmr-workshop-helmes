# Offload

Docker Offload is a fully managed service for building and running containers in the cloud using the Docker tools you already know, including Docker Desktop, the Docker CLI, and Docker Compose. It extends your local development workflow into a scalable, cloud-powered environment, so you can offload compute-heavy tasks, accelerate builds, and securely manage container workloads across the software lifecycle.

![Docker Offload](./.labspace/images/offload.png)

**Starting offload**

```bash
docker offload start
```


**Stopping offload**

```bash
docker offload stop
```

* Use when your PC is not having a powerful GPU, and needs some help
* Offload session is automatically destroyed after 30min of inactivity.
* Behind the scenes - it setups an AWS machine with NVIDIA GPU and 24GB VRAM