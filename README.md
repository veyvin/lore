<!-- markdownlint-disable MD033 MD041 -->
<a id="readme-top"></a>

## Docker 部署指南

### 使用预构建镜像

Lore 提供了预构建的 Docker 镜像，托管在 GitHub Container Registry (ghcr.io)。

**拉取镜像：**

```bash
docker pull ghcr.io/veyvin/lore:latest
```

**运行容器：**

```bash
docker run -d \
  --name lore-server \
  -p 41337:41337 \
  -p 41339:41339 \
  -v lore-data:/data \
  --restart unless-stopped \
  ghcr.io/veyvin/lore:latest
```

**端口说明：**
- `41337`: QUIC + gRPC 端口
- `41339`: HTTP 端口

**数据持久化：**
容器内的 `/data` 目录用于存储 Lore 数据，建议通过数据卷进行持久化。

**访问服务：**
容器启动后，可通过以下地址访问：
- HTTP API: `http://localhost:41339`

### 使用 Docker Compose

创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  lore-server:
    image: ghcr.io/veyvin/lore:latest
    container_name: lore-server
    restart: unless-stopped
    ports:
      - "41337:41337"  # QUIC + gRPC
      - "41339:41339"  # HTTP
    volumes:
      - lore-data:/data
    environment:
      - RUST_LOG=info

volumes:
  lore-data:
    driver: local
```

**启动服务：**

```bash
# 启动容器
docker-compose up -d

# 查看日志
docker-compose logs -f lore-server

# 停止服务
docker-compose down
```

### 编译 Docker 镜像

如果需要自行编译 Docker 镜像，可使用以下命令：

```bash
# 克隆仓库
git clone https://github.com/EpicGames/lore.git
cd lore

# 编译镜像
docker build -f lore-server/Dockerfile -t lore-server:latest .

# 运行自定义镜像
docker run -d \
  --name lore-server \
  -p 41337:41337 \
  -p 41339:41339 \
  -v lore-data:/data \
  --restart unless-stopped \
  lore-server:latest
```

### 在绿联 NAS 上部署

绿联 NAS 支持 Docker 功能，可通过以下方式部署：

1. **启用 Docker 和 SSH**
   - 进入「应用中心」安装 Docker 应用
   - 在「控制面板」→「终端机」启用 SSH

2. **SSH 连接到 NAS**
   ```bash
   ssh admin@你的NAS_IP地址
   ```

3. **拉取并运行镜像**
   ```bash
   docker pull ghcr.io/veyvin/lore:latest
   mkdir -p /docker/lore-data
   docker run -d \
     --name lore-server \
     -p 41337:41337 \
     -p 41339:41339 \
     -v /docker/lore-data:/data \
     --restart unless-stopped \
     ghcr.io/veyvin/lore:latest
   ```

4. **验证运行状态**
   ```bash
   docker ps
   docker logs lore-server
   ```

**注意事项：**
- 确保 41337 和 41339 端口未被占用
- 如遇权限问题，执行 `chmod -R 777 /docker/lore-data`

---

<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/icons/Lore_TM_White_V1.svg">
  <img alt="Lore — open source version control by Epic Games" src="docs/assets/icons/Lore_TM_Black_V1.svg" width="220">
</picture>

<h1>Lore</h1>

<p><strong>Next-generation open source version control</strong></p>

<p>
  <a href="https://github.com/EpicGames/lore/releases">Download Lore</a>
  &nbsp;&middot;&nbsp;
  <a href="https://epicgames.github.io/lore/tutorials/quickstart/">Quickstart</a>
  &nbsp;&middot;&nbsp;
  <a href="https://epicgames.github.io/lore/">Read the docs</a>
  &nbsp;&middot;&nbsp;
  <a href="https://discord.gg/E4SFJKRPbg">Join the conversation</a>
</p>

<p>
  <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-informational"></a>
  <img alt="Built with Rust" src="https://img.shields.io/badge/Built%20with-Rust-orange">
</p>

</div>

<details>
  <summary>Table of contents</summary>

- [About Lore](#about-lore)
- [Get started with Lore](#get-started-with-lore)
- [Overview](#overview)
- [Lore's architecture](#lores-architecture)
- [Lore's repositories](#lores-repositories)
- [Fully open source](#fully-open-source)
- [Contributing](#contributing)
- [License](#license)
- [Contact and community](#contact-and-community)

</details>

## About Lore

Lore is an open source version control system designed for unprecedented scalability of both data and teams. It is optimized for projects that combine code with large binary assets, including games and entertainment, and caters for the needs of developers and artists alike.

> [!NOTE]
> Lore is pre-1.0 and under active development. Interfaces, on-disk formats, and APIs may change between releases.

<sub><a href="#readme-top">(back to top)</a></sub>

## Get started with Lore

- **Quickstart** — install Lore and make your first commit by following the [quickstart guide](https://epicgames.github.io/lore/tutorials/quickstart/).
- **Read the docs** — delve into Lore's ethos and architecture in the [Lore documentation](https://epicgames.github.io/lore/).
- **Have questions?** — the [FAQ](https://epicgames.github.io/lore/faq/) covers licensing, supported platforms, production readiness, and how Lore compares to other version control systems.
- **See where Lore is headed** — the [roadmap](https://epicgames.github.io/lore/roadmap/) lays out the big-rock features by time horizon, from scalable locking to an open source desktop client.
- **Join the conversation** — chat with us and our community on [Discord](https://discord.gg/E4SFJKRPbg).

Or try it right now — install Lore and start a local server in demo mode:

**macOS / Linux**

```bash
curl -fsSL https://raw.githubusercontent.com/EpicGames/lore/main/scripts/install.sh | bash -s -- --demo
```

**Windows (PowerShell)**

```powershell
$env:LORE_DEMO=1; irm https://raw.githubusercontent.com/EpicGames/lore/main/scripts/install.ps1 | iex
```

<sub><a href="#readme-top">(back to top)</a></sub>

## Overview

- **Easy setup, on-demand scalability** — Get started in local mode in minutes. Then, scale up as far and as fast as you need.
- **Fast and efficient processes** — Scale without slowdowns, thanks to shared, reusable data and as-needed downloads.
- **Free branching** — Quickly and easily create, manage, and sync branches to freely experiment, iterate, and release.
- **History you can trust** — Confidently track and manage revisions with Lore's verifiable tamper-evident source of truth.
- **Intuitive interface** — Enjoy complete one-to-one access to the full Lore functionality via the CLI.
- **Full-surface API** — Extend, customize, and integrate Lore via C/C++, C#, Rust, Go, Python, or JavaScript.

> [!NOTE]
> Lore is the built-in version control system for UEFN (Unreal Editor for Fortnite), but today's open source tooling can't yet talk to it: the UEFN build uses a proprietary compression format that can't ship with the open source project. We're actively moving UEFN onto an open compression format — the same one this open source project uses — to eliminate the gap between the two.

<sub><a href="#readme-top">(back to top)</a></sub>

## Lore's architecture

Lore is a centralized, content-addressed version control system that represents repository state as Merkle trees and an immutable revision chain, optimized for binary-first storage, deduplication, and sparse/on-demand data hydration at scale. For the full model—on-disk formats, chunking internals, and the mechanics of the Merkle tree—read the [system design doc](https://epicgames.github.io/lore/explanation/system-design/).

### Highlights

- **Content-addressed storage** — Repository data is stored and referenced by content hash in a Merkle tree, enabling fast comparisons, integrity checks, and reuse across history and branches.
- **Immutable revision chain** — A revision's hash signature is derived from its revision state, including parent revision hashes and contained data hashes, forming an immutable chain with cryptographic integrity.
- **Chunked storage for large files** — Files are stored as reusable chunks with indexed lookup, reducing duplication and enabling efficient updates and transfer for large binary assets.
- **On-demand hydration and sparse workspaces** — Workspaces can stay lightweight by fetching file data only when needed, so you don't have to download everything up front.
- **Centralized service with caching** — A service-backed architecture uses caching in front of durable storage to scale throughput for large teams and repositories.
- **Lightweight branches and fast switching** — Branches are lightweight mutable references, so creating and switching branches is low-overhead without duplication of underlying data.


<sub><a href="#readme-top">(back to top)</a></sub>

## Lore's repositories

Lore spans a family of repositories: the core library, server, and CLI in this repository, plus a software development kit (SDK) for each supported language.

| Repository | Description | Link |
| --- | --- | --- |
| **Lore Library, Server & CLI** | The core Lore library, the Lore Server, and the Lore CLI. You are here. | [View on GitHub](https://github.com/EpicGames/lore) |
| **JavaScript SDK** | The JavaScript binding for the Lore API. | [View on GitHub](https://github.com/EpicGames/lore-js) |
| **Python SDK** | The Python binding for the Lore API. | [View on GitHub](https://github.com/EpicGames/lore-python) |
| **C# SDK** | The C# binding for the Lore API. | [View on GitHub](https://github.com/EpicGames/lore-dotnet) |
| **Go SDK** | The Go binding for the Lore API. | [View on GitHub](https://github.com/EpicGames/lore-go) |

<sub><a href="#readme-top">(back to top)</a></sub>

## Fully open source

We believe a truly open ecosystem is built collectively using open standards. Lore is fully open source under an [MIT license](LICENSE), and we invite you to build the version control system of the future in the open. See [CONTRIBUTING.md](CONTRIBUTING.md) to get involved.

<sub><a href="#readme-top">(back to top)</a></sub>

## Contributing

Contributions of every kind are welcome — code, documentation, bug reports, and reviews. Start with [CONTRIBUTING.md](CONTRIBUTING.md) for the development workflow, then read the [Code of Conduct](CODE_OF_CONDUCT.md) and the project [governance model](GOVERNANCE.md). New to the codebase? The [`good-first-issue`](https://github.com/EpicGames/lore/labels/good-first-issue) label is a good place to start.

<sub><a href="#readme-top">(back to top)</a></sub>

## License

Lore is released under the MIT License. See [LICENSE](LICENSE) for the full text. Copyright (c) 2026 Epic Games, Inc.

<sub><a href="#readme-top">(back to top)</a></sub>

## Contact and community

- **Discord —** chat with the team and community on [Discord](https://discord.gg/E4SFJKRPbg).
- **GitHub Issues —** report bugs and request features through [GitHub Issues](https://github.com/EpicGames/lore/issues).

<sub><a href="#readme-top">(back to top)</a></sub>
