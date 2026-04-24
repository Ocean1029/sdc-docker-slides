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

<v-clicks>

- 包含：程式語言、相依套件、設定檔、環境變數
- 容易做版本控制、開發者間共享
- 開發 / 測試 / 正式環境執行結果一致
- 不用各自手動建置環境

</v-clicks>

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

<v-clicks>

- 同一份程式碼在不同機器上行為不一致
- README 難以窮盡所有環境細節，容易過時
- 開發機 / 測試機 / 正式機是各自獨立的黑盒
- 部署時噴錯，難以重現也難以排查

</v-clicks>

---

# 容器化的適用場景

<v-clicks>

### 多人協作開發

不同 OS、不同版本工具 → 容器確保每人啟動服務時有完全相同的執行環境

### 標準化的專案啟動程序

Dockerfile 作為 Single Source of Truth，取代容易過時的手動安裝說明

### 一致的部署流程

開發、測試、正式環境使用同一個設定流程，降低環境差異造成的問題

</v-clicks>

---

# Docker 是什麼？

Docker 是一個**跑容器的服務**，把容器技術包裝成一套好用的工具鏈和生態系。

<div class="grid grid-cols-2 gap-6 mt-8">

<div v-click>

#### 「包服務」的那一端

開發者寫 Dockerfile 定義環境

→ Docker 建置出 **Image**（映像檔）

→ 把 Image 推到 **Registry**（公共倉庫）

</div>

<div v-click>

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

<v-clicks>

- 每台 VM 都有完整的 **Guest OS**
- Kernel + User Space 都包在裡面
- 可以當作獨立電腦使用
- 常見：VMware、KVM、PVE

</v-clicks>

::right::

# 容器（Container）

利用 Linux 核心本身的功能來隔離程序

<v-clicks>

- **不模擬硬體層**
- 所有容器共用主機 Kernel
- 每個容器只包含 App + 相依函式庫
- 大小幾百 MB
- 適合做服務的部署

</v-clicks>

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

<v-click>

預期輸出：

```
Client:
 Version:           28.5.2
 API version:       1.51 ......
```

看到這段訊息就代表 Docker 裝好了。

</v-click>

<!-- 還沒裝 Docker 的話，請先看根目錄 README 的安裝流程。 -->

---

# 跑第一個容器

```bash
docker run --name first-container hello-world
```

<v-click>

輸出：

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

</v-click>

---

# 跑 Nginx

```bash
docker run --name my-nginx -p 8080:80 nginx:1.27-alpine
```

<v-click>

打開瀏覽器 → `http://localhost:8080`

看到 **404 Not Found** 代表服務啟動起來了。

</v-click>

<v-click>

另一個 terminal 執行 `docker ps`：

```
CONTAINER ID   IMAGE                STATUS           PORTS                  NAMES
7bbb864c2612   nginx:1.27-alpine    Up 14 minutes    0.0.0.0:8080->80/tcp   my-nginx
```

</v-click>

---

# 停止與清理

```bash {1-2|4-6|all}
# 停止容器（或在原 terminal 按 Ctrl+C）
docker stop my-nginx

# 徹底清理
docker rm my-nginx
docker rm first-container
```

<v-click>

> `hello-world` 執行完會自動停止，不用另外 `docker stop`。

</v-click>

---

# Homebrew vs Docker

其實不用 Docker 也能跑 Nginx：`brew install nginx` + `brew services start nginx`

但兩者使用情境不一樣。

| 面向 | Homebrew | Docker |
|------|----------|--------|
| **本質** | 把套件裝進你的系統 | 把應用跑在隔離的容器裡 |
| **影響** | 改變主機 | 不動主機 |
| **多版本** | 通常只能裝一個 | 同時跑多版本互不干擾 |

<v-click>

- **Brew**：每天會叫用的 CLI 工具（`git`、`jq`、`kubectl`、`go`、`node`）
- **Docker**：要被部署、需要隔離的應用服務（`postgres`、`redis`、`kafka`、你自己寫的後端）

</v-click>

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

<v-click>

### `docker run` 時一定要指定 Image

本機沒有 Image 時，Docker 會自動從 Registry pull：

```
Unable to find image 'hello-world:latest' locally
Pulling from library/hello-world
```

</v-click>

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

<v-click>

```bash
# 刪除指定 Image
docker rmi nginx:1.27

# 刪除所有未使用的 Image（dangling）
docker image prune
```

</v-click>

---

# 一張 Image，多個容器

同一張 Image 可以跑出好幾個彼此獨立的容器：

```bash {1|1-2|1-3|all}
docker run -d --name nginx1 -p 8081:80 nginx:1.27-alpine
docker run -d --name nginx2 -p 8082:80 nginx:1.27-alpine
docker run -d --name nginx3 -p 8083:80 nginx:1.27-alpine
```

<v-click>

```
CONTAINER ID   IMAGE               STATUS   PORTS                  NAMES
675eac764576   nginx:1.27-alpine   Up...    0.0.0.0:8083->80/tcp   nginx3
39b94aa0395a   nginx:1.27-alpine   Up...    0.0.0.0:8082->80/tcp   nginx2
99f4a490669c   nginx:1.27-alpine   Up...    0.0.0.0:8081->80/tcp   nginx1
```

> `-d` = detach（背景執行）

</v-click>

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

<v-click>

Image layer 唯讀、可共享；每個 container 各自擁有獨立的可寫層。

</v-click>

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

<v-click>

**常見 Tag 慣例：**

- `nginx:latest` — 最新版（**不建議正式環境使用**）
- `nginx:1.27.3` — 精確版本（正式環境建議使用）
- `nginx:1.27-alpine` — 基於 Alpine 的精簡版

</v-click>

---
layout: section
---

# 1.5 容器基本操作

---

# `docker run` 基本語法

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

<v-clicks>

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

</v-clicks>

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

<v-click>

```
CONTAINER ID   IMAGE               STATUS          PORTS                  NAMES
a1b2c3d4e5f6   nginx:1.27-alpine   Up 10 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

</v-click>

---

# 容器生命週期

```bash {1-2|4-5|7-8|10-11|13-14|all}
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

<v-click>

打開瀏覽器連 `http://localhost:80` → **連不上**

</v-click>

<v-click>

`docker ps` 卻顯示容器正常運行：

```
CONTAINER ID   IMAGE               STATUS          PORTS   NAMES
7bbb864c2612   nginx:1.27-alpine   Up 30 seconds           naughty_brown
```

`PORTS` 欄位是空的 — 容器內的服務沒有對外開放。

</v-click>

---

# 為什麼需要 Port Mapping？

<v-clicks>

- 一台電腦同時跑很多網路服務：Web、DB、SSH……
- 它們共用同一個 IP，作業系統用 **Port**（0–65535）區分流量
- **每個容器有獨立的網路空間（Network Namespace）**
- 容器內部的服務對主機而言是**不可見**的

</v-clicks>

<v-click>

### Port Mapping

在主機與容器之間建立轉發通道，將主機指定 port 收到的流量轉發至容器內的 port。

</v-click>

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

<v-click>

| 用法 | 指令 |
|------|------|
| 對映多個 Port | `docker run -p 8080:80 -p 8443:443 nginx` |
| 限定綁定 IP | `docker run -p 127.0.0.1:8080:80 nginx` |

</v-click>

<v-click>

> 同一台機器可以跑很多容器，只要 **Host Port 不衝突**就沒問題。Container Port 彼此獨立，十個容器都在容器內監聽 3306 也不會撞到。

</v-click>

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

<v-click>

容器停掉後砍掉重跑：

```bash
docker rm db && docker run --name db postgres:17-alpine
```

</v-click>

<v-click>

### 資料全部消失

容器的可寫層（1.4 節）跟容器生命週期綁在一起，`docker rm` 的當下就歸零。

需要一種機制把資料放在容器之外 → **Volume / Bind Mount**

</v-click>

---

# Volume vs Bind Mount

| 類型 | 儲存位置 | 設定方法 |
|------|---------|---------|
| **Bind Mount** | 主機上指定的目錄 | `-v <主機路徑>:<容器路徑>` |
| **Named Volume** | Docker 管理，`/var/lib/docker/volumes/` | `-v <Volume 名稱>:<容器路徑>` |

<v-click>

| 類型 | 範例 | 適用場景 |
|------|-----|---------|
| **Bind Mount** | `-v /home/user/code:/app` | 開發時同步原始碼 |
| **Named Volume** | `-v mydata:/var/lib/mysql` | 資料庫等需要持久化的資料 |

</v-click>

---

# Bind Mount 範例

把目前目錄下的 `html/` 掛進 nginx 容器：

```bash
docker run -d --name web -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:1.27-alpine
```

<v-click>

> `$(pwd)` 是 shell 取目前目錄絕對路徑的語法。
>
> **Docker 要求 Bind Mount 一律用絕對路徑。**

</v-click>

---

# Named Volume 範例

```bash {1|2-3|all}
docker volume create pgdata
docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data postgres:17-alpine
```

<v-click>

> Named Volume 使用前要先用 `docker volume create` 建立，像宣告變數一樣。

</v-click>

---

# Volume 管理

```bash
# 列出所有 Volume
docker volume ls

# 查看 Volume 詳細資訊
docker volume inspect pgdata

# 刪除指定 Volume
docker volume rm pgdata

# 刪除所有未使用的 Volume（執行前請先問你家老大）
docker volume prune
```

---
layout: section
---

# 1.8 容器除錯

---

# 查看日誌

```bash
docker logs my-nginx
```

<v-click>

| 參數 | 效果 | 範例 |
|------|------|------|
| `-f` | 持續追蹤（類似 `tail -f`） | `docker logs -f my-nginx` |
| `--tail N` | 只看最後 N 行 | `docker logs --tail 100 my-nginx` |
| `-t` | 顯示時間戳記 | `docker logs -t my-nginx` |
| `--since` | 指定時間之後 | `docker logs --since 2024-01-01 my-nginx` |

</v-click>

<v-click>

組合使用：

```bash
docker logs -f --tail 50 -t my-nginx
```

</v-click>

---

# 進入容器內部

```bash {1-2|4-5|7-8|all}
# 在容器內開啟 shell
docker exec -it my-nginx sh

# 執行單一命令（不進入互動模式）
docker exec my-nginx cat /etc/nginx/nginx.conf

# 以 root 身份進入
docker exec -it -u root my-nginx sh
```

<v-click>

- `-i` 保持標準輸入開啟（interactive）
- `-t` 分配偽終端（tty）
- 基底是 **Alpine** 的 Image 通常沒有 bash，要用 `sh` 進去

</v-click>

---

# 查看資源使用狀況

```bash
# 即時顯示所有容器的資源使用
docker stats

# 只看特定容器
docker stats my-nginx
```

<v-click>

```
CONTAINER ID   NAME       CPU %   MEM USAGE / LIMIT     MEM %   NET I/O
a1b2c3d4e5f6   my-nginx   0.00%   3.441MiB / 7.667GiB   0.04%   1.45kB / 0B
```

</v-click>

---
layout: section
---

# 1.9 練習 1

## 執行你的第一個容器

---

# 動手做

完成以下練習：

📄 `exercises/01-first-container.md`

<v-click>

練習會涵蓋：

- 啟動一個 Nginx 容器
- 用 Port Mapping 讓瀏覽器連到它
- 掛 Volume 做資料持久化
- 進入容器除錯
- 清理資源

</v-click>

---
layout: end
---

# 第一章結束

下一章：**Docker Compose**

`02-docker-compose.md`
