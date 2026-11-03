# 개발 워크스테이션 구축 미션

## 1. 프로젝트 개요

터미널(리눅스 CLI), Docker, Git/GitHub를 직접 손으로 세팅하고 사용해 보며, 재현 가능한 개발 환경을 만드는 미션입니다.

## 2. 실행 환경

| 항목 | 내용 |
|---|---|
| OS | Ubuntu 26.04.1 LTS (WSL 2, Windows) |
| 쉘/터미널 | bash / VSCode 터미널 |
| Docker | Docker version 29.9.2, build a82e528 |
| Git | git version 2.53.0 |

## 3. 수행 항목 체크리스트

- [x] 터미널 조작 (위치/목록/이동/생성/복사/이동/삭제/내용 확인)
- [x] 파일/디렉토리 권한 실습 (chmod)
- [x] Docker 설치 및 점검 (version, info)
- [x] Docker 기본 운영 (images, ps, logs, stats)
- [x] hello-world 및 ubuntu 컨테이너 실행
- [x] Dockerfile 기반 커스텀 이미지 (nginx:alpine)
- [x] 포트 매핑 (8080:80)
- [x] 바인드 마운트
- [x] Docker 볼륨 영속성
- [x] Git 설정
- [x] GitHub/VSCode 연동

## 4. 검증 방법

각 항목마다 "어떤 명령으로 무엇을 확인했는지"를 정리했습니다.

### 4-1. Docker 설치 및 점검

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| Docker 버전 | `docker --version` | Docker version 29.9.2 출력 |
| Docker 엔진 동작 | `docker info` | Server Version이 출력되어 엔진이 켜져 있음 |

### 4-2. Docker 기본 운영

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| 이미지 받기/목록 | `docker pull hello-world`, `docker images` | hello-world 이미지가 목록에 보임 |
| 컨테이너 실행/목록 | `docker run hello-world`, `docker ps`, `docker ps -a` | 환영 메시지 출력, 종료된 컨테이너는 ps -a에서만 보임 |
| 로그 확인 | `docker logs my-nginx` | nginx 시작 기록이 출력됨 |
| 리소스 확인 | `docker stats --no-stream` | CPU/메모리 사용량 표가 출력됨 |
| 우분투 컨테이너 | `docker run -it --name my-ubuntu ubuntu` | 컨테이너 안에서 ls, echo 실행 성공 |
| 종료/유지 차이 | `docker exec -it my-nginx bash` | exit 후에도 컨테이너가 Up 상태로 유지됨 |

### 4-3. 커스텀 이미지 (nginx:alpine 기반)

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| Dockerfile 작성 | `cat Dockerfile` | FROM, COPY 두 줄 확인 ([app/Dockerfile](app/Dockerfile)) |
| 이미지 빌드 | `docker build -t my-custom-nginx .` | 빌드 성공, docker images에 표시 |
| 컨테이너 실행 | `docker run -d --name my-custom-web my-custom-nginx` | 컨테이너 안에서 내 페이지 응답 확인 |

### 4-4. 포트 매핑

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| 포트 연결 | `docker run -d -p 8080:80 ...`, `curl http://localhost:8080` | 내 컴퓨터 8080 포트로 접속 성공 |
| 브라우저 접속 | 주소창에 `http://localhost:8080` 입력 | 브라우저에서 내 페이지가 표시됨 |

**브라우저 접속 증거 (주소창 `localhost:8080` 포함)**

![포트 매핑 접속 화면](images/port-mapping.png)

### 4-5. 바인드 마운트

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| 변경 전 | `docker run -d -p 8081:80 -v $HOME/bind-site:/usr/share/nginx/html nginx:alpine` | Version 1 (before) 표시 |
| 변경 후 | `echo "<h1>Version 2 (after)</h1>" > ~/bind-site/index.html` | 컨테이너 재시작 없이 Version 2 (after) 표시 |

> 아래 브라우저 증거는 `my-web:1.0` 이미지로 만든 `web-bind` 컨테이너(`-p 8080:80`, `app` 폴더 마운트)로 다시 실습해 찍은 화면입니다. 터미널 로그는 "4-8. Docker 명령 실행 로그"의 바인드 마운트 항목에 있습니다.

**변경 전 (호스트의 `index.html`이 Version 1)**

![바인드 마운트 변경 전](images/bind-mount-before.png)

**변경 후 (컨테이너를 다시 만들지 않고 호스트 파일만 수정 → Version 2 반영)**

![바인드 마운트 변경 후](images/bind-mount-after.png)

### 4-6. Docker 볼륨 영속성

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| 볼륨 생성/연결/데이터 입력 | `docker volume create my-data`, `docker run -d --name vol-test -v my-data:/data alpine sleep 3600` | note.txt에 데이터 저장됨 |
| 컨테이너 삭제 후 확인 | `docker rm -f vol-test`, `docker run --rm -v my-data:/data alpine cat /data/note.txt` | 컨테이너 삭제 후에도 데이터가 남아 있음 |

### 4-7. Git 설정 및 GitHub 연동

| 확인 내용 | 명령어 | 확인한 결과 |
|---|---|---|
| Git 설정 | `git config --list` | user.name, user.email, init.defaultbranch=main 확인 |
| VSCode와 WSL 연결 | VSCode 왼쪽 아래 `><` → Connect to WSL, File → Open Folder | 왼쪽 아래에 WSL: Ubuntu 표시 |
| GitHub 로그인 | VSCode 계정 메뉴 | 내 GitHub 아이디 표시 |
| 저장소 연동 | `git init`, `git remote add origin ...`, `git remote -v`, `git push -u origin main` | origin 주소 확인, GitHub 저장소에 파일 업로드 |

**VSCode의 WSL 연결과 GitHub 로그인 증거**

![VSCode WSL 연결 및 GitHub 로그인](images/vscode-github.png)

> 터미널 조작과 권한 실습의 명령어와 출력은 아래 "5. 터미널 조작 로그"에 기록했습니다.

### 4-8. Docker 명령 실행 로그

**설치 확인 (`docker --version`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker --version
Docker version 29.9.2, build a82e528
```

**엔진 동작 점검 (`docker info`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker info
Client:
 Version:    29.9.2
 Context:    default
 Debug Mode: false
 Plugins:
  agent: Docker AI Agent Runner (Docker Inc.)
    Version:  v1.152.0
    Path:     /usr/local/lib/docker/cli-plugins/docker-agent
  ai: Docker AI Agent - Ask Gordon (Docker Inc.)
    Version:  v1.33.0
    Path:     /usr/local/lib/docker/cli-plugins/docker-ai
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.38.1
    Path:     /usr/local/lib/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v5.6.1
    Path:     /usr/local/lib/docker/cli-plugins/docker-compose
  debug: Get a shell into any image or container (Docker Inc.)
    Version:  0.49.0
    Path:     /usr/local/lib/docker/cli-plugins/docker-debug
  desktop: Docker Desktop commands (Docker Inc.)
    Version:  4.96.3
    Path:     /usr/local/lib/docker/cli-plugins/docker-desktop
  dhi: CLI for managing Docker Hardened Images (Docker Inc.)
    Version:  v0.0.9
    Path:     /usr/local/lib/docker/cli-plugins/docker-dhi
  extension: Manages Docker extensions (Docker Inc.)
    Version:  v0.2.32
    Path:     /usr/local/lib/docker/cli-plugins/docker-extension
  init: Creates Docker-related starter files for your project (Docker Inc.)
    Version:  v1.4.1
    Path:     /usr/local/lib/docker/cli-plugins/docker-init
  mcp: Docker MCP Plugin (Docker Inc.)
    Version:  v0.44.2
    Path:     /usr/local/lib/docker/cli-plugins/docker-mcp
  model: Docker Model Runner (Docker Inc.)
    Version:  v1.2.9
    Path:     /usr/local/lib/docker/cli-plugins/docker-model
  offload: Docker Offload (Docker Inc.)
    Version:  v0.7.19
    Path:     /usr/local/lib/docker/cli-plugins/docker-offload
  pass: Docker Pass Secrets Manager Plugin (beta) (Docker Inc.)
    Version:  v0.3.2
    Path:     /usr/local/lib/docker/cli-plugins/docker-pass
  sandbox: "docker sandbox" is deprecated, use Docker Sandboxes instead (Docker Inc.)
    Version:  v0.13.0
    Path:     /usr/local/lib/docker/cli-plugins/docker-sandbox
  scout: Docker Scout (Docker Inc.)
    Version:  v1.27.0
    Path:     /usr/local/lib/docker/cli-plugins/docker-scout

Server:
 Containers: 3
  Running: 0
  Paused: 0
  Stopped: 3
 Images: 2
 Server Version: 29.9.2
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: local
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Discovered Devices:
  cdi: docker.com/gpu=webgpu
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 nvidia runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b7a25261e029d68978bbf290df0a62fc819b3c51
 runc version: v1.5.2-0-g29dd3dc2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.18.54.1-microsoft-standard-WSL2
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 3.681GiB
 Name: docker-desktop
 ID: 38556747-0678-4710-99b5-127ac2be444a
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 HTTP Proxy: http.docker.internal:3128
 HTTPS Proxy: http.docker.internal:3128
 No Proxy: hubproxy.docker.internal
 Labels:
  com.docker.desktop.address=unix:///var/run/docker-cli.sock
 Experimental: false
 Insecure Registries:
  hubproxy.docker.internal:5555
  127.0.0.0/8
  ::1/128
 Live Restore Enabled: false
 Firewall Backend: iptables
```

**이미지 받기와 목록 (`docker pull`, `docker images`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
Digest: sha256:c567b231f65614688711a75f31518a064258a51d442117a3e771a3a8fdba78ef
Status: Downloaded newer image for hello-world:latest
docker.io/library/hello-world:latest
seoran00@DESKTOP-9LTHJF0:~/practice$ docker images
                                                                                                    i Info →   U  In Use
IMAGE                ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest   c567b231f656       25.9kB         9.49kB    U
ubuntu:latest        c098726dd9b9        163MB         45.9MB    U
```

**hello-world 실행과 컨테이너 목록 (`docker run`, `docker ps`, `docker ps -a`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
seoran00@DESKTOP-9LTHJF0:~/practice$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

seoran00@DESKTOP-9LTHJF0:~/practice$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
seoran00@DESKTOP-9LTHJF0:~/practice$ docker ps -a
CONTAINER ID   IMAGE          COMMAND    CREATED              STATUS                          PORTS     NAMES
37c238433050   hello-world    "/hello"   About a minute ago   Exited (0) About a minute ago             determined_villani
6a29715c3495   ubuntu         "bash"     7 days ago           Exited (137) 7 days ago                   myubuntu
b144a5bfb216   ubuntu         "bash"     7 days ago           Exited (0) 7 days ago                     determined_proskuriakova
a3d897962f35   11a3eb1597e8   "/hello"   7 days ago           Exited (0) 7 days ago                     clever_pascal
```

**nginx 컨테이너 실행 (`docker run -d`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker run -d --name my-nginx nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
a670537bed70: Pull complete
6fc242d221a6: Pull complete
d322f1bf41d8: Pull complete
9f604dae09b2: Pull complete
6e2ab1f3ef4e: Pull complete
a071113ac575: Pull complete
c139fe92686d: Pull complete
05a7d8870b4b: Download complete
ca81da665177: Download complete
Digest: sha256:600f7cd45e27aa13e9304c834ab7c1b0ff39df85ee1dcc5087955455d77bc83b
Status: Downloaded newer image for nginx:latest
10a11dde4662aa1b61571cd02c9bef7a8ed5c96fd020d90212b00216d74a74f0
```

**로그 확인 (`docker logs`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker logs my-nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/10/30 13:53:54 [notice] 1#1: using the "epoll" event method
2026/10/30 13:53:54 [notice] 1#1: nginx/1.31.8
2026/10/30 13:53:54 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19)
2026/10/30 13:53:54 [notice] 1#1: OS: Linux 6.18.54.1-microsoft-standard-WSL2
2026/10/30 13:53:54 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2026/10/30 13:53:54 [notice] 1#1: start worker processes
2026/10/30 13:53:54 [notice] 1#1: start worker process 29
2026/10/30 13:53:54 [notice] 1#1: start worker process 30
2026/10/30 13:53:54 [notice] 1#1: start worker process 31
2026/10/30 13:53:54 [notice] 1#1: start worker process 32
2026/10/30 13:53:54 [notice] 1#1: start worker process 33
2026/10/30 13:53:54 [notice] 1#1: start worker process 34
2026/10/30 13:53:54 [notice] 1#1: start worker process 35
2026/10/30 13:53:54 [notice] 1#1: start worker process 36
```

**리소스 확인 (`docker stats`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker stats --no-stream
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT    MEM %     NET I/O         BLOCK I/O     PIDS
10a11dde4662   my-nginx   0.00%     7.77MiB / 3.681GiB   0.21%     1.17kB / 126B   0B / 12.3kB   9
```

**우분투 컨테이너 실행과 종료/유지 차이 (`docker run -it`, `docker exec`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker run -it --name my-ubuntu ubuntu
root@1165e7187a5a:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@1165e7187a5a:/# echo "hello from ubuntu container"
hello from ubuntu container
root@1165e7187a5a:/# exit
exit
```

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ docker exec -it my-nginx bash
root@10a11dde4662:/# ls
bin   dev                  docker-entrypoint.sh  home  lib64  mnt  proc  run   srv  tmp  var
boot  docker-entrypoint.d  etc                   lib   media  opt  root  sbin  sys  usr
root@10a11dde4662:/# exit
exit
```

**커스텀 이미지 빌드와 실행 (`docker build`, `docker run`)**

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker build -t my-custom-nginx .
[+] Building 22.2s (7/7) FINISHED               docker:default
 => [internal] load build definition from Dockerfile      0.5s
 => => transferring dockerfile: 104B                      0.1s
 => [internal] load metadata for docker.io/library/nginx  5.0s
 => [internal] load .dockerignore                         0.4s
 => => transferring context: 2B                           0.0s
 => [internal] load build context                         0.7s
 => => transferring context: 81B                          0.0s
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:785  8.8s
 => => resolve docker.io/library/nginx:alpine@sha256:785  0.4s
 => => sha256:9b0f7dfe35b2711a17f60dadd6 1.40kB / 1.40kB  0.4s
 => => sha256:b9b388a21ecfde6ac80505f2 20.53MB / 20.53MB  4.0s
 => => sha256:2ec5fb117ffa9a78b15fb6a787 1.21kB / 1.21kB  0.6s
 => => sha256:26185d08a3ec318dd85a6c9f2172e7 404B / 404B  0.6s
 => => sha256:7cc330898115ce55acccb0cc1e7dd0 964B / 964B  0.6s
 => => sha256:657023555a2a19dc6036468198bb35 629B / 629B  0.5s
 => => sha256:12b85cff06fd7d383d36a9c965 1.94MB / 1.94MB  1.2s
 => => sha256:7b229517ed222597277036d948 3.85MB / 3.85MB  1.4s
 => => extracting sha256:7b229517ed222597277036d94892173  0.6s
 => => extracting sha256:12b85cff06fd7d383d36a9c96503fe0  0.5s
 => => extracting sha256:657023555a2a19dc6036468198bb350  0.4s
 => => extracting sha256:7cc330898115ce55acccb0cc1e7dd09  0.3s
 => => extracting sha256:26185d08a3ec318dd85a6c9f2172e71  0.2s
 => => extracting sha256:2ec5fb117ffa9a78b15fb6a78745ffc  0.2s
 => => extracting sha256:9b0f7dfe35b2711a17f60dadd6d4035  0.2s
 => => extracting sha256:b9b388a21ecfde6ac80505f23f52714  0.7s
 => [2/2] COPY index.html /usr/share/nginx/html/index.ht  0.8s
 => exporting to image                                    5.0s
 => => exporting layers                                   1.1s
 => => exporting manifest sha256:c9ac3c510cc5ec0839cb97a  0.2s
 => => exporting config sha256:9cc9c8a94d5c289955b724da0  0.2s
 => => exporting attestation manifest sha256:4935332e0f7  0.5s
 => => exporting manifest list sha256:7fc62c7dc411a8c113  0.2s
 => => naming to docker.io/library/my-custom-nginx:lates  0.0s
 => => unpacking to docker.io/library/my-custom-nginx:la  2.4s
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker images
                                           i Info →   U  In Use
IMAGE          ID             DISK USAGE   CONTENT SIZE   EXTRA
hello-world:latest
               c567b231f656       25.9kB         9.49kB    U
my-custom-nginx:latest
               7fc62c7dc411       93.6MB         26.3MB
nginx:latest   600f7cd45e27        243MB         66.8MB    U
ubuntu:latest
               c098726dd9b9        163MB         45.9MB    U
```

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker run -d --name my-custom-web my-custom-nginx
0d652377ef94f355184a79edc3067644930b345222c024c13e70a0711215cfd6
```

**포트 매핑 (`-p 8080:80`)**

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker rm -f my-custom-web
my-custom-web
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker run -d --name my-custom-web -p 8080:80 my-custom-nginx
d70e11854677b149dbc0bb44bb39ee1a2860c57c0fcc7a361c6c00a200301885
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                                     NAMES
d70e11854677   my-custom-nginx   "/docker-entrypoint.…"   17 seconds ago   Up 15 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-custom-web
seoran00@DESKTOP-9LTHJF0:~/my-web$ curl http://localhost:8080
<h1>Hello from my custom Docker image!</h1>
```

**바인드 마운트 (변경 전/후)**

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ mkdir ~/bind-site
seoran00@DESKTOP-9LTHJF0:~/my-web$ echo "<h1>Version 1 (before)</h1>" > ~/bind-site/index.html
seoran00@DESKTOP-9LTHJF0:~/my-web$ cat ~/bind-site/index.html
<h1>Version 1 (before)</h1>
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker run -d --name my-bind-web -p 8081:80 -v $HOME/bind-site:/usr/share/nginx/html nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
Digest: sha256:785835ea65db4bba6f021acc0f3be91b0fef76a53fe41f8588c6698d2aed0606
Status: Downloaded newer image for nginx:alpine
99c7986c86f62a1f6b3fa684a52038cd5aab8bc4a3a6a6b07ef26e0618816237
seoran00@DESKTOP-9LTHJF0:~/my-web$ curl http://localhost:8081
<h1>Version 1 (before)</h1>
```

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ echo "<h1>Version 2 (after)</h1>" > ~/bind-site/index.html
seoran00@DESKTOP-9LTHJF0:~/my-web$ cat ~/bind-site/index.html
<h1>Version 2 (after)</h1>
seoran00@DESKTOP-9LTHJF0:~/my-web$ curl http://localhost:8081
<h1>Version 2 (after)</h1>
```

**Docker 볼륨 (컨테이너 삭제 전/후)**

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker volume create my-data
my-data
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker volume ls
DRIVER    VOLUME NAME
local     my-data
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker run -d --name vol-test -v my-data:/data alpine sleep 3600
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
7b229517ed22: Already exists
3da1513d2124: Download complete
3ec36f3670df: Download complete
Digest: sha256:be2656061b536c1883c4034790e21d0a09c5aaaf51968b89368a3c0371ea2c9a
Status: Downloaded newer image for alpine:latest
271fa580ba61112872a6838820ca7ea63feebbe09b4fe0fa2be19abe5c14703c
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker exec vol-test sh -c "echo 'I survive container deletion' > /data/note.txt"
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker exec vol-test cat /data/note.txt
I survive container deletion
```

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker rm -f vol-test
vol-test
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS                     PORTS                                     NAMES
99c7986c86f6   nginx:alpine      "/docker-entrypoint.…"   16 minutes ago   Up 16 minutes              0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-bind-web
d70e11854677   my-custom-nginx   "/docker-entrypoint.…"   34 minutes ago   Up 34 minutes              0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-custom-web
1165e7187a5a   ubuntu            "/bin/bash"              21 hours ago     Exited (0) 21 hours ago                                              my-ubuntu
10a11dde4662   nginx             "/docker-entrypoint.…"   21 hours ago     Exited (255) 2 hours ago   80/tcp                                    my-nginx
37c238433050   hello-world       "/hello"                 21 hours ago     Exited (0) 21 hours ago                                              determined_villani
6a29715c3495   ubuntu            "bash"                   8 days ago       Exited (137) 8 days ago                                              myubuntu
b144a5bfb216   ubuntu            "bash"                   8 days ago       Exited (0) 8 days ago                                                determined_proskuriakova
a3d897962f35   11a3eb1597e8      "/hello"                 8 days ago       Exited (0) 8 days ago                                                clever_pascal
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker volume ls
DRIVER    VOLUME NAME
local     my-data
seoran00@DESKTOP-9LTHJF0:~/my-web$ docker run --rm -v my-data:/data alpine cat /data/note.txt
I survive container deletion
```

**Git 설정 (`git config --list`)**

```text
seoran00@DESKTOP-9LTHJF0:~/my-web$ git config --global user.name "seoran00"
seoran00@DESKTOP-9LTHJF0:~/my-web$ git config --global user.email "****"
seoran00@DESKTOP-9LTHJF0:~/my-web$ git config --global init.defaultBranch main
seoran00@DESKTOP-9LTHJF0:~/my-web$ git config --list
user.name=seoran00
user.email=****
init.defaultbranch=main
```

## 5. 터미널 조작 로그

> 처음 시도할 때 남긴 로그는 [logs/first-attempt-log.md](logs/first-attempt-log.md)에 보관했습니다.

### 5-1. 터미널 기본 조작

**현재 위치 확인 (`pwd`)**

```text
seoran00@DESKTOP-9LTHJF0:~$ pwd
/home/seoran00
```

**목록 확인, 숨김 파일 포함 (`ls -a`)**

```text
seoran00@DESKTOP-9LTHJF0:~$ ls -a
.  ..  .aws  .azure  .bash_history  .bash_logout  .bashrc  .cache  .config  .docker  .motd_shown  .profile  practice
```

**폴더 만들기, 이동 (`mkdir`, `cd`)**

```text
seoran00@DESKTOP-9LTHJF0:~$ mkdir practice
mkdir: practice: File exists
seoran00@DESKTOP-9LTHJF0:~$ cd practice
seoran00@DESKTOP-9LTHJF0:~/practice$ pwd
/home/seoran00/practice
```

**빈 파일 만들기, 내용 쓰기, 내용 확인 (`touch`, `echo`, `cat`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ touch empty.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ echo "hello linux" > hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ ls
empty.txt  folder1  hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ cat hello.txt
hello linux
```

**복사, 이름 바꾸기, 이동 (`cp`, `mv`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ cp hello.txt copy.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ mv copy.txt renamed.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ mkdir folder1
mkdir: folder1: File exists
seoran00@DESKTOP-9LTHJF0:~/practice$ mv renamed.txt folder1/
seoran00@DESKTOP-9LTHJF0:~/practice$ ls
empty.txt  folder1  hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ ls folder1
renamed.txt
```

**삭제 (`rm`)**

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ rm empty.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ mkdir temp_dir
seoran00@DESKTOP-9LTHJF0:~/practice$ rm -r temp_dir
seoran00@DESKTOP-9LTHJF0:~/practice$ ls
folder1  hello.txt
```

### 5-2. 권한 실습

**파일 권한 (`hello.txt`)**

변경 전:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -l hello.txt
-rw-r--r-- 1 seoran00 seoran00 12 Oct 30 18:42 hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -l
total 8
drwxr-xr-x 2 seoran00 seoran00 4096 Oct 30 18:49 folder1
-rw-r--r-- 1 seoran00 seoran00   12 Oct 30 18:42 hello.txt
```

`chmod 000`으로 변경 후:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ chmod 000 hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -l hello.txt
---------- 1 seoran00 seoran00 12 Oct 30 18:42 hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ cat hello.txt
cat: hello.txt: Permission denied
```

`chmod 644`로 되돌린 후:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ chmod 644 hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -l hello.txt
-rw-r--r-- 1 seoran00 seoran00 12 Oct 30 18:42 hello.txt
seoran00@DESKTOP-9LTHJF0:~/practice$ cat hello.txt
hello linux
```

**폴더 권한 (`folder1`)**

변경 전:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -ld folder1
drwxr-xr-x 2 seoran00 seoran00 4096 Oct 30 18:49 folder1
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -l folder1
total 4
-rw-r--r-- 1 seoran00 seoran00 12 Oct 30 18:46 renamed.txt
```

`chmod 000`으로 변경 후:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ chmod 000 folder1
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -ld folder1
d--------- 2 seoran00 seoran00 4096 Oct 30 18:49 folder1
seoran00@DESKTOP-9LTHJF0:~/practice$ cd folder1
-bash: cd: folder1: Permission denied
```

`chmod 755`로 되돌린 후:

```text
seoran00@DESKTOP-9LTHJF0:~/practice$ chmod 755 folder1
seoran00@DESKTOP-9LTHJF0:~/practice$ ls -ld folder1
drwxr-xr-x 2 seoran00 seoran00 4096 Oct 30 18:49 folder1
seoran00@DESKTOP-9LTHJF0:~/practice$ cd folder1
seoran00@DESKTOP-9LTHJF0:~/practice/folder1$ pwd
/home/seoran00/practice/folder1
seoran00@DESKTOP-9LTHJF0:~/practice/folder1$ cd ..
seoran00@DESKTOP-9LTHJF0:~/practice$
```

## 6. 개념 정리

### 6-1. 절대 경로와 상대 경로

- **절대 경로**: 맨 앞 `/`(루트)부터 시작하는 전체 주소입니다. 어디서 입력해도 같은 곳을 가리킵니다.
  - 예: `/home/seoran00/practice/hello.txt`
- **상대 경로**: 지금 있는 위치를 기준으로 쓰는 주소입니다. `..`은 한 단계 위 폴더입니다.
  - 예: 현재 위치가 `/home/seoran00/practice`일 때 `hello.txt`, `folder1/renamed.txt`
- 같은 파일을 가리키는 두 가지 방법:
  - 절대 경로: `cat /home/seoran00/practice/hello.txt`
  - 상대 경로: `cat hello.txt` (practice 폴더 안에서)

### 6-2. 파일 권한 (r/w/x, 755, 644)

- `r`(read, 읽기)=4, `w`(write, 쓰기)=2, `x`(execute, 실행)=1이며, 합쳐서 숫자 하나로 씁니다.
- 권한은 **주인 / 그룹 / 그 외 사람** 세 부류에게 순서대로 줍니다.
- `644` = 주인 `rw-`(4+2), 그룹 `r--`(4), 그 외 `r--`(4) → 주인만 수정 가능
- `755` = 주인 `rwx`(4+2+1), 그룹 `r-x`(4+1), 그 외 `r-x`(4+1) → 주인만 수정 가능, 모두 실행/진입 가능
- 폴더에서는 `x`가 "그 폴더 안으로 들어가기(cd)"를 뜻합니다. 실습에서 `chmod 000 folder1` 후 `cd folder1`이 Permission denied로 막히는 것을 확인했습니다.

### 6-3. 커스텀 이미지 (기존 베이스 활용)

- **선택한 기존 베이스**: `nginx:alpine` (웹 서버 베이스 이미지 활용 방식)
- **커스텀 포인트와 목적**
  - `FROM nginx:alpine`: 웹 서버가 이미 들어 있고 용량이 작아서 선택했습니다.
  - `COPY index.html /usr/share/nginx/html/index.html`: 기본 화면 대신 내가 만든 페이지를 보여주기 위해 교체했습니다.
- 소스: [app/Dockerfile](app/Dockerfile), [app/index.html](app/index.html)

### 6-4. 포트 매핑이 필요한 이유

- 컨테이너는 격리된 환경이라 밖(내 브라우저)에서 바로 접속할 수 없습니다.
- `-p 8080:80`은 "내 컴퓨터의 8080번 포트로 들어오면 컨테이너의 80번 포트로 연결"해서, 브라우저에서 `http://localhost:8080`으로 접속할 수 있게 합니다.

### 6-5. 이미지와 컨테이너, 종료/유지 차이

- **이미지**는 컨테이너를 만들기 위한 설계도(읽기 전용), **컨테이너**는 그 이미지로 실제 실행되는 인스턴스입니다. 같은 이미지로 컨테이너를 여러 개 만들 수 있습니다.
- `docker run -it ubuntu`로 들어갔다가 `exit`하면 컨테이너도 함께 종료(Exited)됐습니다. 컨테이너의 메인 프로그램이 내가 쓰던 bash였기 때문입니다.
- 이미 실행 중인 컨테이너에 `docker exec -it my-nginx bash`로 들어갔다가 `exit`해도 컨테이너는 계속 실행(Up)됐습니다. 메인 프로그램(nginx)은 그대로 돌고 있었기 때문입니다.

### 6-6. 바인드 마운트와 Docker 볼륨

| 구분 | 바인드 마운트 | Docker 볼륨 |
|---|---|---|
| 저장 위치 | 내가 정한 내 컴퓨터 폴더 | Docker가 관리하는 저장소 |
| 특징 | 내 컴퓨터에서 파일을 고치면 컨테이너에 바로 반영 | 컨테이너를 삭제해도 데이터가 남음(영속성) |
| 실습 | `-v $HOME/bind-site:/usr/share/nginx/html` | `-v my-data:/data` |
| 용도 | 개발할 때 코드 수정 반영 | 데이터 보관 |

### 6-7. Git과 GitHub의 차이

- **Git**: 내 컴퓨터(로컬)에서 파일의 변경 기록을 관리하는 버전 관리 도구입니다.
- **GitHub**: Git으로 만든 기록을 인터넷(원격)에 올려 보관하고, 다른 사람과 협업하는 플랫폼입니다.

## 7. 트러블슈팅

### 7-1. `docker info`에서 `permission denied ... docker.sock`

- **문제**: `docker info` 실행 시 `Server:` 아래에 `permission denied while trying to connect to the docker API at unix:///var/run/docker.sock`가 나왔다.
- **원인 가설**: 내 사용자(`seoran`)가 `docker` 그룹에 없어서 Docker 소켓에 접근할 권한이 없는 것 같다.
- **확인**: `which docker`는 `/usr/bin/docker`였고, `groups` 결과(`seoran adm cdrom sudo dip plugdev users`)에 `docker`가 없었다.
- **해결**: `sudo usermod -aG docker $USER`로 docker 그룹에 추가하고, `exit` 후 PowerShell에서 `wsl --shutdown`으로 WSL을 재시작한 뒤 우분투를 다시 열었다. 이후 `docker info`, `docker run hello-world`가 정상 동작했다.

### 7-2. `git push`에서 비밀번호 요구, 이어서 403 오류

- **문제**: `git push -u origin main`에서 `Password for 'https://...@github.com'`을 물었고, 토큰을 넣은 뒤에는 `Permission to ... denied`, `error: 403`이 나왔다.
- **원인 가설**: GitHub는 터미널에서 비밀번호 인증을 지원하지 않아 토큰(Personal Access Token)이 필요하고, 처음 만든 토큰에 `repo` 권한이 부족했던 것 같다.
- **확인**: 오류 메시지의 `403`(권한 없음)을 보고, 로그인은 통과했지만 쓰기 권한에서 막혔다고 판단했다. 저장소 주소(`seoran00/E1-1`)가 내 계정인지, 토큰에 `repo`가 체크되어 있는지 확인했다.
- **해결**: GitHub의 Settings → Developer settings에서 Personal access token(classic)을 `repo` 권한으로 만들어 Password 칸에 입력했다. 인증 오류는 사라지고 다음 오류(7-9)로 넘어갔다. (토큰은 README, 스크린샷에 남기지 않았다.)

### 7-3. `git push`에서 `rejected ... (fetch first)`

- **문제**: `! [rejected] main -> main (fetch first)`와 "the remote contains work that you do not have locally" 메시지가 나왔다.
- **원인 가설**: GitHub 저장소를 만들 때 README 등이 함께 생성되어, 원격에 내 컴퓨터에 없는 커밋이 있는 것 같다.
- **확인**: 오류 메시지의 `remote contains work that you do not have locally` 문장.
- **해결**: `git pull origin main --rebase`로 원격 내용을 가져와 합친 뒤 `git push -u origin main`으로 다시 올렸다.
