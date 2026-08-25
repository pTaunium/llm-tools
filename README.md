# Unofficial Auto-Packaged GitHub Releases

This repository contains automated GitHub Actions workflows that monitor specific upstream open-source repositories for new releases, and automatically package their pre-compiled Linux amd64 binaries into Docker images. 

The resulting images are pushed to the GitHub Container Registry (ghcr.io) and act as lightweight **data carriers**.

## 📦 How it works

These Docker images are built `FROM scratch` and simply contain the unmodified `.tar.gz` release asset from the upstream repositories. 

This approach is highly useful for environments like Kubernetes, where you can use these images as `InitContainers` to copy the binaries into a shared volume before the main application starts, without needing a full OS environment just to distribute the files.

## 🛠️ Packaged Tools

Currently, this repository tracks and packages the following tools:

| Tool | Upstream Repository | License |
| :--- | :--- | :--- |
| **rtk** | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) | [Apache 2.0](https://github.com/rtk-ai/rtk/blob/main/LICENSE) |
| **codebase-memory-mcp** | [DeusData/codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) | [MIT](https://github.com/DeusData/codebase-memory-mcp/blob/main/LICENSE) |

## ⚠️ Disclaimer

**This is an unofficial project.** 
The Docker images provided here are NOT officially maintained or endorsed by the original authors of `rtk` or `codebase-memory-mcp`. 

* All source code, trademarks, and intellectual property remain with their respective original authors.
* Please refer to the upstream repositories linked above for official documentation, issue tracking, and source code.
* The original `LICENSE` files are included within the packaged `.tar.gz` archives inside the Docker images.

## 🚀 Usage Examples

Because these images use `scratch` as a base and contain a compressed archive, they cannot be run directly. Here are two common ways to extract and use the data:

### Option A: Extract directly in Linux via Docker CLI

If you just want to download and extract the archive to your local Linux machine using Docker:

```bash
# 1. Create a dummy container from the image
CONTAINER_ID=$(docker create ghcr.io/pTaunium/rtk:v1.0.0)

# 2. Copy the archive out to your current directory
docker cp $CONTAINER_ID:/rtk.tar.gz ./

# 3. Clean up the dummy container
docker rm $CONTAINER_ID

# 4. Extract the archive
tar -xzf rtk.tar.gz
```

### Option B: Use in a Multi-stage Docker Build

If you are building your own Docker image and want to include the tools:

```Dockerfile
FROM ubuntu:22.04

# Copy the archive from the data carrier image
COPY --from=ghcr.io/pTaunium/rtk:v1.0.0 /rtk.tar.gz /tmp/

# Extract and install
RUN tar -xzf /tmp/rtk.tar.gz -C /usr/local/bin/ \
    && rm /tmp/rtk.tar.gz
```

