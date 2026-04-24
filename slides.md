---
theme: default
title: Docker 核心概念與基本操作
info: |
  ## SDC SRE Workshop — Docker 篇
  Day 1 · 容器化基礎
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
---

# Docker 核心概念與基本操作

SDC SRE Workshop · Day 1

<div class="pt-12">
  <span class="text-sm opacity-60">按空白鍵或右方向鍵開始 →</span>
</div>

---
layout: section
---

# 1.1 什麼是容器化？

---

# 容器化（Containerization）

將「應用程式」與「完整執行環境」封裝成可移植的元件 → **容器（Container）**

- 包含：程式語言、相依套件、設定檔、環境變數
- 容易做版本控制、開發者間共享
- 開發 / 測試 / 正式環境執行結果一致
- 不用各自手動建置環境

<!--
開場比喻：把應用程式跟它需要的東西一起「打包」。
強調：同一包東西，到哪裡跑結果都一樣。
-->

---

# 沒有容器化的情境

不同開發者的環境可能長這樣：

| 開發者 | 作業系統 | Go 版本 | PostgreSQL | 函式庫 |
|--------|---------|---------|-----------|--------|
| A | macOS   | 1.21    | 15        | v2.3.1 |
| B | Ubuntu  | 1.19    | 14        | v2.1.0 |

- 同一份程式碼在不同機器上行為不一致
- README 難以窮盡所有環境細節，容易過時
- 開發機 / 測試機 / 正式機是各自獨立的黑盒
- 部署時噴錯，難以重現也難以排查

---

# 容器化的適用場景

### 多人協作開發

不同 OS、不同版本工具 → 容器確保每人啟動服務時有完全相同的執行環境

### 標準化的專案啟動程序

Dockerfile 作為 Single Source of Truth，取代容易過時的手動安裝說明

### 一致的部署流程

開發、測試、正式環境使用同一個設定流程，降低環境差異造成的問題

---

# Docker 是什麼？

Docker 是一個**跑容器的服務**，把容器技術包裝成一套好用的工具鏈和生態系。

<div class="grid grid-cols-2 gap-6 mt-8">

<div>

#### 「包服務」的那一端

開發者寫 Dockerfile 定義環境

→ Docker 建置出 **Image**（映像檔）

→ 把 Image 推到 **Registry**（公共倉庫）

</div>

<div>

#### 「用服務」的那一端

從 Registry 把 Image pull 到本機

→ 用 Docker 把 **Container** 跑起來

→ 開始使用別人包好的服務

</div>

</div>

---
layout: section
---

# 1.2 容器 vs 虛擬機器

---
layout: two-cols
---

# 虛擬機器（VM）

透過 **Hypervisor** 在硬體上模擬出多台虛擬主機

- 每台 VM 都有完整的 **Guest OS**
- Kernel + User Space 都包在裡面
- 可以當作獨立電腦使用
- 常見：VMware、KVM、PVE

::right::

# 容器（Container）

利用 Linux 核心本身的功能來隔離程序

- **不模擬硬體層**
- 所有容器共用主機 Kernel
- 每個容器只包含 App + 相依函式庫
- 大小幾百 MB
- 適合做服務的部署

---

# 架構比較

```
┌──────────────────────────────┬──────────────────────────────┐
│        虛擬機器 (VM)          │        容器 (Container)       │
├──────────────────────────────┼──────────────────────────────┤
│  ┌────────┐  ┌────────┐      │  ┌────────┐  ┌────────┐      │
│  │ App A  │  │ App B  │      │  │ App A  │  │ App B  │      │
│  ├────────┤  ├────────┤      │  ├────────┤  ├────────┤      │
│  │ Bins/  │  │ Bins/  │      │  │ Bins/  │  │ Bins/  │      │
│  │ Libs   │  │ Libs   │      │  │ Libs   │  │ Libs   │      │
│  ├────────┤  ├────────┤      │  └────┬───┘  └───┬────┘      │
│  │Guest OS│  │Guest OS│      │  ┌────┴──────────┴─────┐     │
│  └────┬───┘  └───┬────┘      │  │   Docker Engine     │     │
│  ┌────┴──────────┴────┐      │  ├─────────────────────┤     │
│  │    Hypervisor      │      │  │     Host OS         │     │
│  ├────────────────────┤      │  ├─────────────────────┤     │
│  │     Host OS        │      │  │    Infrastructure   │     │
│  ├────────────────────┤      │  └─────────────────────┘     │
│  │   Infrastructure   │      │                              │
│  └────────────────────┘      │                              │
└──────────────────────────────┴──────────────────────────────┘
```

---

# 比較表

| 比較項目 | 虛擬機器 (VM) | 容器 (Container) |
|---------|--------------|-----------------|
| **隔離方式** | 硬體層虛擬化（Hypervisor） | OS 層虛擬化（Namespace + Cgroup） |
| **Guest OS** | 每台 VM 都有完整 OS | 共用主機核心，無 Guest OS |
| **映像檔大小** | GB 級 | MB 級 |
| **資源佔用** | 高 | 低 |
| **部署密度** | 數台至數十台 | 數十至數百個 |
| **適用場景** | 強隔離、不同 OS、遺留系統 | 微服務、CI/CD、快速部署 |

---
layout: section
---

# 1.3 用 Docker 啟動服務

---

# 驗證安裝

```bash
docker version
```

預期輸出：

```
Client:
 Version:           28.5.2
 API version:       1.51 ......
```

看到這段訊息就代表 Docker 裝好了。

<!-- 還沒裝 Docker 的話，請先看根目錄 README 的安裝流程。 -->

---

# 跑第一個容器

```bash
docker run --name first-container hello-world
```

輸出：

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

# 跑 Nginx

```bash
docker run --name my-nginx -p 8080:80 nginx:1.27-alpine
```

打開瀏覽器 → `http://localhost:8080`

看到 **404 Not Found** 代表服務啟動起來了。

另一個 terminal 執行 `docker ps`：

```
CONTAINER ID   IMAGE                STATUS           PORTS                  NAMES
7bbb864c2612   nginx:1.27-alpine    Up 14 minutes    0.0.0.0:8080->80/tcp   my-nginx
```

---

# GUI 版：查看容器

<div class="flex justify-center mt-2">
  <img src="/orbstack-container-info.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Containers 頁，相當於 `docker ps` + `docker inspect`

</div>

---

# 停止與清理

```bash
# 停止容器（或在原 terminal 按 Ctrl+C）
docker stop my-nginx

# 徹底清理
docker rm my-nginx
docker rm first-container
```

> `hello-world` 執行完會自動停止，不用另外 `docker stop`。

---

# Homebrew vs Docker

其實不用 Docker 也能跑 Nginx：`brew install nginx` + `brew services start nginx`

但兩者使用情境不一樣。

| 面向 | Homebrew | Docker |
|------|----------|--------|
| **本質** | 把套件裝進你的系統 | 把應用跑在隔離的容器裡 |
| **影響** | 改變主機 | 不動主機 |
| **多版本** | 通常只能裝一個 | 同時跑多版本互不干擾 |

- **Brew**：每天會叫用的 CLI 工具（`git`、`jq`、`kubectl`、`go`、`node`）
- **Docker**：要被部署、需要隔離的應用服務（`postgres`、`redis`、`kafka`、你自己寫的後端）

---
layout: section
---

# 1.4 映像檔（Image）

---

# 什麼是 Image？

一個包含**完整執行環境**的檔案：

- 作業系統基礎檔案
- 執行環境
- 程式碼 + 相依套件
- 設定檔

### `docker run` 時一定要指定 Image

本機沒有 Image 時，Docker 會自動從 Registry pull：

```
Unable to find image 'hello-world:latest' locally
Pulling from library/hello-world
```

---

# Docker Hub — Image 版的 GitHub

預設 registry：`hub.docker.com`

大量官方與社群維護的 Image：`nginx`、`postgres`、`redis`、`node` ……

<div class="mt-6 flex justify-center">
  <img src="/docker-hub-nginx.png" class="h-64 rounded shadow" />
</div>

---

# 管理本機 Image

```bash
docker images
```

```
REPOSITORY                  TAG       IMAGE ID       CREATED         SIZE
sre-workshop-capstone-app   latest    f059fe7ba768   20 hours ago    17.3MB
ardge/devcontainer-base     26.04.2   6c59207e643a   2 weeks ago     3.07GB
ardge/devcontainer-base     26.04.1   548e41b4cc13   2 weeks ago     3.06GB
```

```bash
# 刪除指定 Image
docker rmi nginx:1.27

# 刪除所有未使用的 Image（dangling）
docker image prune
```

---

# GUI 版：Image 清單

<div class="flex justify-center mt-2">
  <img src="/orbstack-images.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Images 頁，相當於 `docker images` + `docker inspect`

</div>

---

# 一張 Image，多個容器

同一張 Image 可以跑出好幾個彼此獨立的容器：

```bash
docker run -d --name nginx1 -p 8081:80 nginx:1.27-alpine
docker run -d --name nginx2 -p 8082:80 nginx:1.27-alpine
docker run -d --name nginx3 -p 8083:80 nginx:1.27-alpine
```

```
CONTAINER ID   IMAGE               STATUS   PORTS                  NAMES
675eac764576   nginx:1.27-alpine   Up...    0.0.0.0:8083->80/tcp   nginx3
39b94aa0395a   nginx:1.27-alpine   Up...    0.0.0.0:8082->80/tcp   nginx2
99f4a490669c   nginx:1.27-alpine   Up...    0.0.0.0:8081->80/tcp   nginx1
```

> `-d` = detach（背景執行）

---

# Image 的分層架構

Image 由好幾個 **Layer** 疊起來，每一層記錄一次變更：

```
┌─────────────────────────────────────────────┐
│  ┌──────────────────────────────────┐       │
│  │  Container Layer（可讀寫）         │      │
│  │  新建檔案、修改設定、日誌           │        │
│  │  ⚠ 容器刪除時，這一層就消失了！       │      │
│  ├──────────────────────────────────┤       │
│  │  Layer 3: Nginx 啟動設定（唯讀）    │      │
│  ├──────────────────────────────────┤ Image │
│  │  Layer 2: 安裝 Nginx（唯讀）       │ Layer │
│  ├──────────────────────────────────┤       │
│  │  Layer 1: Alpine Linux（唯讀）    │       │
│  └──────────────────────────────────┘       │
└─────────────────────────────────────────────┘
```

Image layer 唯讀、可共享；每個 container 各自擁有獨立的可寫層。

---

# Image 命名與 Tag

```
[registry/][username/]repository[:tag]
```

例：`docker.io/library/nginx:1.27-alpine`

| 部分 | 說明 |
|------|------|
| Registry | 預設 `docker.io`，通常省略 |
| Username | 官方 Image 為 `library`，通常省略 |
| Repository | Image 名稱 |
| Tag | 版本標記，省略時為 `latest` |

**常見 Tag 慣例：**

- `nginx:latest` — 最新版（**不建議正式環境使用**）
- `nginx:1.27.3` — 精確版本（正式環境建議使用）
- `nginx:1.27-alpine` — 基於 Alpine 的精簡版

---
layout: section
---

# 1.5 容器基本操作

---

# `docker run` 基本語法

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

```bash
# 前景執行（Ctrl+C 可停止）
docker run nginx:1.27-alpine
```

```bash
# 指定容器名稱
docker run --name my-nginx nginx:1.27-alpine
```

```bash
# 執行後自動刪除（適合一次性任務）
docker run --rm nginx:1.27-alpine nginx -v
```

---

# 常用 `docker run` 參數

| 參數 | 說明 | 範例 |
|------|------|------|
| `-d` | 背景執行（detach） | `docker run -d nginx` |
| `--name` | 命名容器 | `docker run --name web nginx` |
| `-p` | Port mapping | `docker run -p 8080:80 nginx` |
| `-v` | 掛載 Volume | `docker run -v /data:/app/data nginx` |
| `-e` | 設定環境變數 | `docker run -e DB_HOST=db nginx` |
| `--rm` | 停止後自動刪除 | `docker run --rm nginx` |
| `--network` | 指定網路 | `docker run --network=mynet nginx` |

---

# 列出容器

```bash
# 執行中的容器
docker ps

# 所有容器（含已停止）
docker ps -a

# 只顯示容器 ID
docker ps -q
```

```
CONTAINER ID   IMAGE               STATUS          PORTS                  NAMES
a1b2c3d4e5f6   nginx:1.27-alpine   Up 10 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

---

# 容器生命週期

```bash
# 停止（SIGTERM，優雅關閉）
docker stop my-nginx

# 啟動已停止的容器
docker start my-nginx

# 刪除已停止的容器
docker rm my-nginx

# 強制刪除執行中的容器
docker rm -f my-nginx

# 刪除所有已停止的容器
docker container prune
```

---
layout: section
---

# 1.6 Port Mapping

---

# 沒有 `-p` 會怎樣？

```bash
docker run nginx:1.27-alpine
```

打開瀏覽器連 `http://localhost:80` → **連不上**

`docker ps` 卻顯示容器正常運行：

```
CONTAINER ID   IMAGE               STATUS          PORTS   NAMES
7bbb864c2612   nginx:1.27-alpine   Up 30 seconds           naughty_brown
```

`PORTS` 欄位是空的 — 容器內的服務沒有對外開放。

---

# 為什麼需要 Port Mapping？

- 一台電腦同時跑很多網路服務：Web、DB、SSH……
- 它們共用同一個 IP，作業系統用 **Port**（0–65535）區分流量
- **每個容器有獨立的網路空間（Network Namespace）**
- 容器內部的服務對主機而言是**不可見**的

### Port Mapping

在主機與容器之間建立轉發通道，將主機指定 port 收到的流量轉發至容器內的 port。

---

# Host Port vs Container Port

| 名詞 | 說明 |
|------|------|
| **Container Port** | 應用在容器內監聽的 port，僅存在於容器的網路空間 |
| **Host Port** | 主機對外開放的 port，Docker 監聽後轉發至對應 Container Port |

<div class="mt-4 flex justify-center">
  <img src="/port-mapping.png" class="h-64 rounded shadow" />
</div>

---

# Port Mapping 語法與範例

```bash
# -p <Host Port>:<Container Port>
docker run --name web -p 8080:80 nginx:1.27-alpine
```

| 用法 | 指令 |
|------|------|
| 對映多個 Port | `docker run -p 8080:80 -p 8443:443 nginx` |
| 限定綁定 IP | `docker run -p 127.0.0.1:8080:80 nginx` |

> 同一台機器可以跑很多容器，只要 **Host Port 不衝突**就沒問題。Container Port 彼此獨立，十個容器都在容器內監聽 3306 也不會撞到。

---
layout: section
---

# 1.7 資料持久化

---

# 容器砍掉 = 資料消失

假設用 Docker 跑 Postgres 並匯入一份重要資料：

```bash
docker run --name db postgres:17-alpine
# 匯入資料...
```

容器停掉後砍掉重跑：

```bash
docker rm db && docker run --name db postgres:17-alpine
```

### 資料全部消失

容器的可寫層（1.4 節）跟容器生命週期綁在一起，`docker rm` 的當下就歸零。

需要一種機制把資料放在容器之外 → **Volume / Bind Mount**

---

# Volume vs Bind Mount

| 類型 | 儲存位置 | 設定方法 |
|------|---------|---------|
| **Bind Mount** | 主機上指定的目錄 | `-v <主機路徑>:<容器路徑>` |
| **Named Volume** | Docker 管理，`/var/lib/docker/volumes/` | `-v <Volume 名稱>:<容器路徑>` |

| 類型 | 範例 | 適用場景 |
|------|-----|---------|
| **Bind Mount** | `-v /home/user/code:/app` | 開發時同步原始碼 |
| **Named Volume** | `-v mydata:/var/lib/mysql` | 資料庫等需要持久化的資料 |

---

# Bind Mount 範例

把目前目錄下的 `html/` 掛進 nginx 容器：

```bash
docker run -d --name web -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:1.27-alpine
```

> `$(pwd)` 是 shell 取目前目錄絕對路徑的語法。
>
> **Docker 要求 Bind Mount 一律用絕對路徑。**

---

# Named Volume 範例

```bash
docker volume create pgdata
docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data postgres:17-alpine
```

> Named Volume 使用前要先用 `docker volume create` 建立，像宣告變數一樣。

---

# Volume 管理

```bash
# 列出所有 Volume
docker volume ls

# 查看 Volume 詳細資訊
docker volume inspect pgdata

# 刪除指定 Volume
docker volume rm pgdata

# 刪除所有未使用的 Volume（不可逆，請先確認）
docker volume prune
```

---

# GUI 版：Volume 清單

<div class="flex justify-center mt-2">
  <img src="/orbstack-volumes.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Volumes 頁，相當於 `docker volume ls` + `inspect`

</div>

---
layout: section
---

# 1.8 容器除錯

---

# 查看日誌

```bash
docker logs my-nginx
```

| 參數 | 效果 | 範例 |
|------|------|------|
| `-f` | 持續追蹤（類似 `tail -f`） | `docker logs -f my-nginx` |
| `--tail N` | 只看最後 N 行 | `docker logs --tail 100 my-nginx` |
| `-t` | 顯示時間戳記 | `docker logs -t my-nginx` |
| `--since` | 指定時間之後 | `docker logs --since 2024-01-01 my-nginx` |

組合使用：

```bash
docker logs -f --tail 50 -t my-nginx
```

---

# GUI 版：Logs

<div class="flex justify-center mt-2">
  <img src="/orbstack-container-logs.png" class="max-h-[24rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Logs 頁，相當於 `docker logs -f`

</div>

---

# 進入容器內部

```bash
# 在容器內開啟 shell
docker exec -it my-nginx sh

# 執行單一命令（不進入互動模式）
docker exec my-nginx cat /etc/nginx/nginx.conf

# 以 root 身份進入
docker exec -it -u root my-nginx sh
```

- `-i` 保持標準輸入開啟（interactive）
- `-t` 分配偽終端（tty）
- 基底是 **Alpine** 的 Image 通常沒有 bash，要用 `sh` 進去

---

# GUI 版：Terminal

<div class="flex justify-center mt-2">
  <img src="/orbstack-container-terminal.png" class="max-h-[22rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Terminal 頁，相當於 `docker exec -it <container> sh`

</div>

---

# GUI 版：Files

<div class="flex justify-center mt-2">
  <img src="/orbstack-container-files.png" class="max-h-[22rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Files 頁 — 不用進 exec 就能瀏覽容器內檔案系統

</div>

---

# 查看資源使用狀況

```bash
# 即時顯示所有容器的資源使用
docker stats

# 只看特定容器
docker stats my-nginx
```

```
CONTAINER ID   NAME       CPU %   MEM USAGE / LIMIT     MEM %   NET I/O
a1b2c3d4e5f6   my-nginx   0.00%   3.441MiB / 7.667GiB   0.04%   1.45kB / 0B
```

---

# GUI 版：Stats

<div class="flex justify-center mt-2">
  <img src="/orbstack-container-stats.png" class="max-h-[22rem] rounded shadow" />
</div>

<div class="text-sm opacity-70 mt-4 text-center">

OrbStack 的 Stats 頁 — 相當於 `docker stats`，附 CPU / Memory / Network / Disk 即時圖表

</div>

---
layout: section
---

# 1.9 練習 1

## 執行你的第一個容器

---

# 動手做

完成以下練習：

📄 `exercises/01-first-container.md`

練習會涵蓋：

- 啟動一個 Nginx 容器
- 用 Port Mapping 讓瀏覽器連到它
- 掛 Volume 做資料持久化
- 進入容器除錯
- 清理資源

---
layout: section
---

# 第二章
## Docker Compose

多容器編排

---
layout: section
---

# 2.1 為什麼需要 Docker Compose？

---

# 多容器的痛點

實際專案幾乎都是多個容器組成：Web app + DB + Cache + ……

- 每跑一個服務就要打一串 `docker run`
- 參數又臭又長，順序不能錯
- 沒有單一檔案能 commit 進 Git
- 新人加入要一條一條傳 SOP

<!--
Snow 問 Ocean：「指令記在哪？」
「...我記在腦袋裡。」
「喔不。」
-->

---

# Docker Compose 是什麼？

Docker 官方提供的**多容器編排工具**

- 用一個 YAML 設定檔定義所有容器（Image / Port / Volume / Env / 相依）
- 一個指令 `docker compose up` 完成整個應用部署
- 設定檔可以納入版本控制
- OrbStack 已內建，可用 `docker compose version` 確認

---

# 範例：PostgreSQL + Adminer

`postgres` 是資料庫；`adminer` 是輕量的網頁資料庫管理介面。

```
┌──────────────────────────────────────────────────────┐
│                  PostgreSQL + Adminer                │
├──────────────────────────────────────────────────────┤
│                   ┌──────────┐                       │
│                   │  Client  │                       │
│                   └────┬─────┘                       │
│                        │ http://localhost:8080       │
│                        ▼                             │
│              ┌──────────────────┐                    │
│              │     Adminer      │  ← Container 1     │
│              │  (DB Admin UI)   │                    │
│              │   (Port 8080)    │                    │
│              └────────┬─────────┘                    │
│                       │  db:5432                     │
│                       ▼                              │
│              ┌──────────────────┐                    │
│              │   PostgreSQL     │  ← Container 2     │
│              │   (Port 5432)    │                    │
│              └──────────────────┘                    │
└──────────────────────────────────────────────────────┘
```

---

# 不用 Compose 的版本

```bash
# 建立網路
docker network create myapp

# 啟動 PostgreSQL
docker run -d --name db --network myapp \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17-alpine

# 啟動 Adminer
docker run -d --name adminer --network myapp \
  -p 8080:8080 \
  -e ADMINER_DEFAULT_SERVER=db \
  adminer:4
```

> 三條指令必須按順序、flag 散在好幾行、沒有單一檔案能 commit 進 Git

---

# 三條指令分別在做什麼

- **`docker network create myapp`**：開一個 bridge 網路，下面兩個容器要加進來才能互通
- **Postgres**：`--name db` 同時是容器名稱也是 DNS 名稱、`-e` 設密碼、`-v` 把資料放進 Named Volume
- **Adminer**：`-p 8080:8080` 對外開 port、`-e ADMINER_DEFAULT_SERVER=db` 預先填好「預設資料庫主機 = `db` 容器」

> 所有 flag 第一章都看過。差別只在這次有兩個容器要互相找到對方。

---

# 用 Compose 的版本

```yaml
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  adminer:
    image: adminer:4
    ports:
      - "8080:8080"
    environment:
      ADMINER_DEFAULT_SERVER: db
    depends_on:
      - db

volumes:
  pgdata:
```

執行：`docker compose up -d`

---

# Adminer 登入畫面

打開 `http://localhost:8080`，系統選 PostgreSQL、使用者填 `postgres`、密碼填 `secret`。

<div class="mt-4 flex justify-center">
  <img src="/adminer-login.png" class="h-72 rounded shadow" />
</div>

---

# YAML 欄位拆解

最上層的 `services` 區塊定義所有容器，每個 key 是一個服務名稱。

| 欄位 | 對應的 docker run flag |
|------|---------------------|
| `image` | `docker run <image>` |
| `ports` | `-p` |
| `environment` | `-e` |
| `volumes` | `-v` |
| `depends_on` | （Compose 專有，控制啟動順序） |

YAML 看起來像 docker run 的「擺正、可重複」版本。

---

# `depends_on` 的陷阱

```yaml
depends_on:
  - db
```

只等容器**啟動**，不等服務真正**準備好**。

- PostgreSQL 容器啟動後還需要幾秒初始化
- 這段時間 Adminer 連過去會直接噴錯

---

# 搭配 healthcheck

```yaml
adminer:
  depends_on:
    db:
      condition: service_healthy

db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
```

- `pg_isready` 是 PostgreSQL 內建的檢查工具
- Compose 每 5 秒問一次「資料庫好了沒」
- 連續失敗 5 次才判定不健康
- `adminer` 會等到 db 的 healthcheck 通過才啟動

---

# 常用 Compose 指令

```bash
# 啟動所有服務（背景）
docker compose up -d

# 看服務狀態
docker compose ps

# 停止並移除所有容器
docker compose down

# 看某個 service 的日誌
docker compose logs <service_name>

# 進入容器的 shell
docker compose exec <service_name> sh
```

> 所有指令都是 `docker compose` 開頭，後面接動作。

---
layout: section
---

# 2.2 Network

---

# 容器間通訊

回顧 1.6：

- 容器網路是隔離的
- 外面連不進去要靠 Port Mapping
- **容器跟容器之間預設也是連不通的**

要讓多個容器互通：把它們放進同一個 **Docker Network**

> Docker Network 是一條虛擬網路線，只有接在同一條線上的容器才能互相找到對方。

---

# Compose 自動建網路

不用 Compose 時：手動 `docker network create myapp`，每個容器加 `--network myapp`

用 Compose 時：**完全不用寫網路設定**

- Compose 會自動建立一個預設網路
- `docker-compose.yml` 裡所有 service 都被放進去
- 容器之間就能互相連線

---

# 服務名稱 = hostname

```yaml
environment:
  ADMINER_DEFAULT_SERVER: db
```

Adminer 用 `db` 當 hostname 連資料庫，Compose 內建的 DNS 自動解析到 PostgreSQL 容器的 IP。

換成 API 連同一個 DB：

```yaml
environment:
  DATABASE_URL: postgres://postgres:secret@db:5432/mydb
```

`@db` 也是同樣的道理。服務改名字，這裡也要跟著改。

> ⚠ 這個 DNS 只在 Compose 的虛擬網路裡有效。在主機上 `ping db` 不會通。

---
layout: section
---

# 2.3 環境變數管理

---

# 把密碼寫進 docker-compose.yml？

Ocean 把 2.1 的 compose 整份 commit 上 public repo，三分鐘後收到 GitHub Secret Scanning 的警告信。Snow：「噢不。」

```yaml
environment:
  POSTGRES_PASSWORD: secret   # ← 會跟著 Git 一起推上去
```

開發時這樣寫沒問題，但 commit 進 Git 等於把密碼公開。

**敏感資訊 → 抽到 `.env`，加進 `.gitignore`**

---

# `.env` 檔案做法

跟 `docker-compose.yml` 放同一個目錄：

```bash
# .env
POSTGRES_USER=myuser
POSTGRES_PASSWORD=secret
POSTGRES_DB=mydb
```

`docker-compose.yml` 裡用 `${var}` 引用：

```yaml
services:
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

Compose 會自動讀同目錄下的 `.env`。

---

# 用 env_file 一次載入

如果環境變數很多，一個一個寫 `${...}` 太麻煩：

```yaml
services:
  adminer:
    env_file:
      - .env
```

把整個 `.env` 一口氣灌進容器。

> ⚠ 記得把 `.env` 加進 `.gitignore`！

---
layout: section
---

# 第三章
## Dockerfile 基礎

把自己的程式打包成 Image

---
layout: section
---

# 3.1 什麼是 Dockerfile？

---

# 自己的程式怎麼上 Image？

前面用的 `nginx`、`postgres` 都是別人放在 Docker Hub 上的 Image。

> 「別人的映像檔當然沒有我們的程式碼」  ── Andrew

要把自己寫的程式變成 Image → 寫一個 **Dockerfile**

- 放在專案根目錄的純文字檔
- 描述 Image 怎麼建：基礎環境、複製哪些檔案、執行什麼指令、程式怎麼啟動
- 寫好之後用 `docker build` 變成 Image，再用 `docker run` 跑起來

---

# 完整循環

```
Dockerfile ──→ docker build ──→ Image ──→ docker run ──→ Container
（建置腳本）                     （成品）                  （跑起來）
```

第一章看的是後半段（用別人的 Image 跑容器）；這章補上前半段（怎麼把自己的程式變成 Image）。

---
layout: section
---

# 3.2 第一個 Dockerfile

---

# 最簡單的範例

建一個空目錄並切進去：

```bash
mkdir hello-docker && cd hello-docker
```

建立檔名叫 `Dockerfile` 的純文字檔：

```dockerfile
FROM alpine:3.21
CMD ["echo", "Hello from my first Dockerfile!"]
```

兩行：基底 Alpine、啟動時印一句話。

---

# build & run

```bash
# 建立 Image，命名為 hello
docker build -t hello .
```

> `.` 是 build context，告訴 Docker 從目前目錄找 Dockerfile 與要打包的檔案。

```bash
# 確認 Image 真的建出來了
docker images hello
```

```
IMAGE          ID             DISK USAGE   CONTENT SIZE
hello:latest   c2de8808511c       8.17MB             0B
```

```bash
# 跑起來
docker run --rm hello
```

```
Hello from my first Dockerfile!
```

---
layout: section
---

# 3.3 Go 網頁服務的 Dockerfile

---

# 範例專案

`exercises/my-app/` 是一個最簡單的 Go HTTP server：

```
exercises/my-app/
├── main.go         # Go 程式碼
├── go.mod          # Go 模組定義
└── Dockerfile      # 建置映像檔的腳本
```

`main.go` 監聽 8080，提供：

- `GET /` → `Hello from Go! Hostname: <hostname>`
- `GET /health` → `OK`

---

# 不用 Docker 怎麼跑？

在一台新機器上跑這個服務，大概要打：

```bash
brew install go                 # 1. 安裝 Go
mkdir /app && cd /app           # 2. 切到工作目錄
git clone <repo-url> .          # 3. 把程式碼放進去
go build -o server .            # 4. 編譯
./server                        # 5. 執行
```

Dockerfile 的目的：把這 5 個步驟寫成檔案，讓 Docker 在乾淨的環境裡照做。

---

# Dockerfile 對照

```dockerfile
FROM golang:1.24-alpine          # 1. 在已經裝好 Go 的環境
WORKDIR /app                     # 2. 切到工作目錄
COPY . .                         # 3. 把程式碼放進去
RUN go build -o server .         # 4. 編譯
CMD ["./server"]                 # 5. 容器啟動時執行
```

五個指令對應五個步驟。

---

# 一行一行拆

- **`FROM golang:1.24-alpine`** — 基礎環境。每個 Dockerfile 的第一行
- **`WORKDIR /app`** — 設定 Image 內部的工作目錄；後面的 `COPY`、`RUN` 都會在這裡執行
- **`COPY . .`** — 把本機目前目錄下所有檔案複製到 Image 的 `/app`
  - 第一個 `.` 是本機路徑
  - 第二個 `.` 是容器內路徑（`WORKDIR` 設定的 `/app`）
- **`RUN go build -o server .`** — 建置時執行命令；結果會寫進 Image 的 Layer
- **`CMD ["./server"]`** — 容器啟動時的預設命令

> `RUN` 在 build 時執行；`CMD` 在容器啟動時執行。

---

# Build & Run

```bash
cd exercises/my-app
docker build -t my-app .
docker run -p 8080:8080 my-app
```

打開瀏覽器連 `http://localhost:8080`：

```
Hello from Go! Hostname: <container-id>
```

---

# Dockerfile 指令速查表

| 指令 | 說明 | 範例 |
|------|------|------|
| `FROM` | 指定基礎 Image（必須是第一行） | `FROM golang:1.24-alpine` |
| `WORKDIR` | 設定工作目錄 | `WORKDIR /app` |
| `COPY` | 複製檔案至 Image | `COPY . .` |
| `RUN` | 建置時執行命令 | `RUN go build -o server .` |
| `CMD` | 容器啟動時的預設命令 | `CMD ["./server"]` |
| `ENTRYPOINT` | 容器啟動時的固定入口 | `ENTRYPOINT ["./server"]` |
| `EXPOSE` | 宣告容器監聽的 port（僅文件用途） | `EXPOSE 8080` |
| `ENV` | 設定環境變數 | `ENV GIN_MODE=release` |
| `USER` | 指定執行使用者 | `USER nonroot` |
| `HEALTHCHECK` | 定義健康檢查 | `HEALTHCHECK CMD curl -f localhost/` |

> AI 很會寫這種東西，對 SRE 而言這部分不是最重要的。

---

# Dockerfile 的價值

- 把專案啟動流程寫成「規範化語言」
- 啟動流程可以維持統一
- 減少新進開發者的設定負擔

完整循環：

```
Dockerfile → docker build → Image → docker run → Container
```

從寫程式到跑起來，都被一個檔案描述清楚。

---
layout: section
---

# 綜合練習

---

# 動手做

完成以下練習：

📄 `exercises/02-push-to-dockerhub.md`

把 my-app 的 Image 推上 Docker Hub，讓別人也能跑你的服務。

---
layout: end
---

# Docker 篇結束

下午：**CI/CD + Prometheus**

謝謝大家！
