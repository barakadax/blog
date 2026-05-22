# Containerization
- [What is a Container?](#what-is-a-container)
- [Why We Need Containers](#why-we-need-containers)
- [History of Containers](#history-of-containers)
- [Example of a Container Configuration File](#example-of-a-container-configuration-file)
- [Structure and Reserved Keywords in Configuration Files](#structure-and-reserved-keywords-in-configuration-files)
- [Where Can Containers Run?](#where-can-containers-run)
- [Best Practices](#best-practices)
  - [Slim Images](#slim-images)
  - [Multi-Stage Builds](#multi-stage-builds)
  - [Least Privilege](#least-privilege)
  - [Pinning Image Tags](#pinning-image-tags)
  - [Hardened Images](#hardened-images)
  - [Handling Signals](#handling-signals)
  - [Mounting Secrets vs. Environment Variables](#mounting-secrets-vs-environment-variables)
- [Complementary Tools](#complementary-tools)
- [Sources](#sources)

## What is a Container?

A **container** is a lightweight, standalone package containing an application and everything needed to run it (code, runtime, system tools, libraries, and settings).
Unlike virtual machines, which virtualize physical hardware and include a full guest OS, containers share the host operating system's kernel.
This makes them highly efficient, fast to start, and resource-friendly.
**Docker** and **Podman** are the most prominent containerization platforms today.

## Why We Need Containers

In traditional software development, setting up execution environments is a major source of friction, often causing the *"it works on my machine"* issue.

Containers resolve this by:
- **Ensuring Parity**: Packaging applications with their exact dependencies and configuration to eliminate environment drift.
- **Simplifying Deployment**: Providing a single, consistent container image that runs identically from development to production.
- **Enhancing Isolation**: Isolating applications from one another and the host system, improving security and resource allocation.

## History of Containers

Containerization is the result of decades of Linux kernel and Unix isolation improvements:

- **1979 (chroot)**: Introduced filesystem isolation by restricting a process's root directory.
- **2000 (FreeBSD Jails)**: Added isolation for processes, users, networks, and filesystems.
- **2002–2013 (Linux Namespaces)**: Virtualized system resources (PID, network, mount, etc.) so processes in one namespace cannot see or affect others.
- **2006 (cgroups)**: Developed by Google to limit, isolate, and monitor hardware resource usage (CPU, memory, disk I/O) for groups of processes.
- **2008 (LXC)**: Combined namespaces and cgroups into the first complete container solution on Linux.
- **2013 (Docker)**: Popularized containers by introducing layered filesystems (UnionFS), the Dockerfile build process, and Docker Hub for easy sharing.
- **2015–Present (OCI & Daemonless Runtimes)**: Standardized container formats via the Open Container Initiative (OCI), enabling daemonless, rootless runtimes like Podman.

## Example of a Container Configuration File

Below is an example of a configuration file (such as a `Dockerfile` or `Containerfile`) for a Rust application:

```dockerfile
FROM rust:latest

ENV IP 0.0.0.0:50051
ENV RUST_BACKTRACE 1

RUN apt-get update && \
    apt-get install -y protobuf-compiler

WORKDIR /app

COPY . .

CMD ["cargo", "run"]
```

## Structure and Reserved Keywords in Configuration Files

Configuration files are step-by-step instructions for assembling a container image. Common instructions include:

- **`FROM`**: Sets the base image (e.g., `FROM rust:latest`).
- **`ENV`**: Defines environment variables that persist during build and runtime.
- **`RUN`**: Executes commands during build time (e.g., installing packages). Creates a new image layer.
- **`WORKDIR`**: Sets the working directory for subsequent instructions.
- **`COPY`**: Copies local files/folders from the host into the container filesystem.
- **`ADD`**: Similar to `COPY`, but can download remote URLs and extract tar archives. *`COPY` is preferred for predictability.*
- **`CMD`**: Sets the default command to run when the container starts. Can be overridden at runtime.
- **`ENTRYPOINT`**: Sets the primary executable. Arguments passed at runtime append to it. Often combined with `CMD` for default arguments.
- **`EXPOSE`**: Documents the container's runtime network ports (does not publish them).
- **`VOLUME`**: Defines mount points for persistent or shared data.

## Where Can Containers Run?

Once a container image is built, it can be run in various execution environments depending on scaling, infrastructure, and traffic needs:

### Local Development
- **When to use**: During development, local debugging, and integration testing.
- **Tools**: Docker/Podman Desktop, Docker/Podman CLI  or Docker/Podman Compose (for orchestrating multi-container local setups).

### Single Host / Virtual Private Server (VPS)
- **When to use**: For simple applications, low-traffic APIs, demo systems, or staging environments.
- **Implementation**: Running Docker or Podman directly on a VM (e.g., AWS EC2, DigitalOcean Droplet) and using tools like systemd to manage container health and startups.

### Container-as-a-Service (CaaS) / Serverless Containers
- **When to use**: For hosting web applications, APIs, or event-driven tasks without the operational overhead of provisioning or managing servers.
Ideal for applications with variable workloads.
- **Implementation**: AWS Fargate, Google Cloud Run, Azure Container Instances (ACI).

### Container Orchestrators
- **When to use**: For production microservices, high-traffic architectures, and distributed systems requiring automatic scaling, self-healing, rolling deployments, and cross-node networking.
- **Implementation**: **Kubernetes** (managed services like EKS, GKE, AKS) or **HashiCorp Nomad**.

## Best Practices

### Slim Images
Use small base images to reduce download times, storage usage, and security risks.
- *Example*: Use `rust:slim` instead of `rust:latest`.

### Multi-Stage Builds
Build applications in one environment and copy only the final artifact to a minimal runtime image.
- *Example*: Compile a Rust application using `rust:latest`, then copy the compiled binary into a bare `debian:stable-slim` image, leaving behind all compilation tools.

### Least Privilege
By default, containers run as `root`. Run processes as a non-root user to mitigate potential security exploits.
- *Example*:
  ```dockerfile
  RUN groupadd -r appuser && useradd -r -g appuser appuser
  USER appuser
  ```

### Pinning Image Tags
Do not use mutable tags like `latest`. Pin specific versions or use cryptographic SHA-256 digests to ensure reproducible builds.
- *Example*: `rust:1.80.0-slim@sha256:0861191076afc8e2dfcf0bec6ad6c2dec8494b3a1e9249729e1989690afed5ec`

### Hardened Images
Use base images containing only the minimal assets required to run the application.
These exclude package managers and shell utilities, decreasing the attack surface.

### Handling Signals
Linux uses signals to send a program something going to happen to it for example stop, kill or background.
Ensure your application responds to signals.
For environments that run as PID 1 and fail to forward signals like Node.js and Python, use a lightweight init process like `tini` or `dumb-init` that wraps your program and handle the signals for them.

### Mounting Secrets vs. Environment Variables
Avoid passing sensitive keys, or passwords via environment variables, as they can be easily leaked.
Mount secrets as read-only files at runtime instead.

## Complementary Tools

- **Hadolint**: A Dockerfile linter that checks configurations against best practices, identifying optimization and security issues.
- **Dive**: A command-line tool for exploring container images. It analyzes image layers and displays their contents, helping identify and remove wasted filesystem space.

## Sources

- [Docker Documentation - What is a Container?](https://www.docker.com/resources/what-container/)
- [Podman Documentation](https://podman.io/docs)
- [Docker Hub Registry](https://hub.docker.com/)
- [Dive - Container Image Analyzer](https://github.com/wagoodman/dive)
- [Dumb-init - Minimal Init Daemon](https://github.com/Yelp/dumb-init)
- [Tini - A Tiny Init for Containers](https://github.com/krallin/tini)
