# 코디세이 E1-1 

> 터미널(CLI), Docker(컨테이너), Git/GitHub(버전 관리) 활용 및 증적.

---

## 📋 실행 환경

| 항목 | 내용 |
|------|------|
| OS | macOS (OrbStack 기반 Docker 환경) |
| Host | `yhkwon.net4691@c3r6s7` |
| Path | `/Users/yhkwon.net4691/work/codyssey-e1-1` |
| Shell | zsh |
| Docker | 27.x (OrbStack) |
| Git | 2.x |

간단한 작업 파일과 문제, 풀이 내용에 대한 파일들은 최상단 디렉토리에 위치하였고, 실습을 위한 index.html은 src 디렉토리 하위에 위치하도록 하였습니다.
```bash
yhkwon.net4691@c3r6s7 codyssey-e1-1 % find . -maxdepth 2 # 현재 디렉토리 구조 조회
.
./dockerfile # 도커 예제 실행을 위한 dockerfile
./E1-1.md # E1-1 문제 파일, 문제를 빠르게 확인하며 진행하기 위해 최상단 디렉토리에 위치
./README.md # 문제 풀이 파일
./src/index.html # hello world codyssey 출력하는 간단한 html 소스, 예제를 위한 소스코드라 src 폴더 하위에 위치시킴
```

---

## ✅ 수행 항목 체크리스트

- [x] 터미널 기본 조작 및 폴더 구성
- [x] 파일 권한 변경 실습 (chmod)
- [x] Docker 설치 및 기본 명령어 수행
- [x] Dockerfile 기반 커스텀 이미지 제작 및 배포
- [x] 포트 매핑 및 볼륨 영속성 검증
- [x] *(Bonus)* Docker Compose 단일/멀티 서비스 운영
- [x] *(Bonus)* 환경 변수를 활용한 설정 분리
- [x] *(Bonus)* GitHub SSH 인증 설정 및 연동

---

## 📂 Step 1 — 터미널 및 권한 실습

터미널 CLI를 통한 파일 조작(pwd, ls, touch, cp, mv, rm)과 리눅스 권한 시스템(chmod)을 진행했습니다.

## 절대 경로 vs 상대 경로

| 구분 | 절대 경로 | 상대 경로 |
|------|-----------|-----------|
| **기준** | 루트(`/`)부터 시작 | 현재 위치 기준 |
| **예시** | `/home/user/project/file.txt` | `./file.txt`, `../config/` |
| **특징** | 어디서 실행해도 항상 동일 | 실행 위치에 따라 달라짐 |

---

### 언제 선택하는가?

**절대 경로를 쓸 때**
- 스크립트/서비스가 **어느 위치에서 실행될지 모를 때** (cron, systemd 등)
- Docker, CI/CD 설정처럼 **환경이 고정되어 있을 때**
- 헷갈리면 안 되는 **시스템 설정 파일** 경로

**상대 경로를 쓸 때**
- 프로젝트 내부에서 **파일끼리 참조할 때** (코드 import, HTML 리소스 등)
- **어디서든 clone해서 쓸 수 있어야 할 때** (이식성)
- 팀 협업 프로젝트에서 **내 로컬 경로를 하드코딩하면 안 될 때**

---

> **요약:** 실행 위치가 고정 → 절대 경로 / 프로젝트가 이동해도 동작해야 함 → 상대 경로

```bash
# 현재 디렉토리 위치 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % pwd
/Users/yhkwon.net4691/work/codyssey-e1-1

# 초기 목록 확인 (권한 포함)
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ls -al
total 32
drwxr-xr-x   5 yhkwon.net4691  yhkwon.net4691    160  4  6 15:16 .
drwxr-xr-x   3 yhkwon.net4691  yhkwon.net4691     96  4  6 15:15 ..
drwxr-xr-x  13 yhkwon.net4691  yhkwon.net4691    416  4  6 15:15 .git
-rw-r--r--   1 yhkwon.net4691  yhkwon.net4691  13335  4  6 15:15 E1-1.md
-rw-r--r--   1 yhkwon.net4691  yhkwon.net4691      0  4  6 15:16 README.md

# 파일 생성 및 복사
yhkwon.net4691@c3r6s7 codyssey-e1-1 % touch test.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % cp test.txt copy.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ls
copy.txt        E1-1.md         README.md       test.txt

# 이름 변경 및 내용 추가 (Redirect)
yhkwon.net4691@c3r6s7 codyssey-e1-1 % mv copy.txt move.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % echo "hi world" >> move.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % cat move.txt
hi world

# 파일 삭제 및 권한 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % rm move.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ls -al test.txt
-rw-r--r--  1 yhkwon.net4691  yhkwon.net4691  0  4  6 16:34 test.txt

# 권한 변경 실습 (644 → 755)
yhkwon.net4691@c3r6s7 codyssey-e1-1 % chmod 755 test.txt
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ls -l test.txt
-rwxr-xr-x  1 yhkwon.net4691  yhkwon.net4691  0  4  6 16:34 test.txt
```

> **설명:** `test.txt`에 실행 권한(`x`)을 추가하여 일반 파일에서 실행 가능 파일 속성으로 변경했습니다.

### 파일 권한 표기 원리 (8진수 & 이진수)

리눅스 권한은 **owner / group / others** 3개 그룹으로 나뉘며, 각 그룹은 `r(4) w(2) x(1)` 3비트로 구성됩니다.

| 권한 | 이진수 | 8진수 | 의미 |
|------|--------|-------|------|
| `rwx` | `111` | `7` | 읽기 + 쓰기 + 실행 |
| `rw-` | `110` | `6` | 읽기 + 쓰기 |
| `r-x` | `101` | `5` | 읽기 + 실행 |
| `r--` | `100` | `4` | 읽기만 |
| `---` | `000` | `0` | 권한 없음 |

```
chmod 644  →  110 100 100  →  rw-r--r--  (변경 전)
chmod 755  →  111 101 101  →  rwxr-xr-x  (변경 후)
              ↑↑↑ ↑↑↑ ↑↑↑
            owner grp others
```

- **644**: owner는 읽기+쓰기(`6`), group/others는 읽기만(`4`) — 일반 텍스트 파일의 기본값
- **755**: owner는 읽기+쓰기+실행(`7`), group/others는 읽기+실행(`5`) — 스크립트·실행 파일의 기본값

---

## 🐳 Step 2 — Docker 기본 및 커스텀 이미지

기본적인 개념과 docker 명령(run, info, build 등) 및 컨테이너 관리 증적

## Docker 이미지, 컨테이너 개념 및 차이

**이미지**는 읽기 전용 템플릿이고, **컨테이너**는 그 이미지를 실행한 인스턴스.

| 관점 | 이미지 | 컨테이너 |
|------|--------|----------|
| **빌드** | `Dockerfile`로 생성, 레이어 구조로 저장, 불변(immutable) | 빌드 대상 아님, 실행 대상 |
| **실행** | 실행 불가, `docker run`의 재료 | 이미지 위에 쓰기 레이어를 올려 실행, 동일 이미지로 여러 개 동시 실행 가능 |
| **변경** | 수정 불가 | 내부 변경은 쓰기 레이어에만 반영, 삭제 시 변경 사항 소멸 → 유지하려면 `docker commit` 또는 볼륨 사용 |

> **요약:** 이미지 = 레시피, 컨테이너 = 요리. 레시피는 그대로, 요리는 삭제하면 없어짐.

```bash
# Docker 버전 및 상태 점검
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker --version
Docker version 28.5.2, build ecc6942
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker info
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker info
Client:
 Version:    28.5.2
 Context:    orbstack
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.29.1
    Path:     /Users/yhkwon.net4691/.docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.40.3
    Path:     /Users/yhkwon.net4691/.docker/cli-plugins/docker-compose

Server:
 Containers: 2
  Running: 1
  Paused: 0
  Stopped: 1
 Images: 2
 Server Version: 28.5.2
 Storage Driver: overlay2
  Backing Filesystem: btrfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
 CDI spec directories:
  /etc/cdi
  /var/run/cdi
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 1c4457e00facac03ce1d75f7b6777a7a851e5c41
 runc version: d842d7719497cc3b774fd71620278ac9e17710e0
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
  cgroupns
 Kernel Version: 6.17.8-orbstack-00308-g8f9c941121b1
 Operating System: OrbStack
 OSType: linux
 Architecture: x86_64
 CPUs: 6
 Total Memory: 15.67GiB
 Name: orbstack
 ID: 93126f77-7b93-4c0e-b834-c17c3707b57a
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  ::1/128
  127.0.0.0/8
 Live Restore Enabled: false
 Product License: Community Engine
 Default Address Pools:
   Base: 192.168.97.0/24, Size: 24
   Base: 192.168.107.0/24, Size: 24
   Base: 192.168.117.0/24, Size: 24
   Base: 192.168.147.0/24, Size: 24
   Base: 192.168.148.0/24, Size: 24
   Base: 192.168.155.0/24, Size: 24
   Base: 192.168.156.0/24, Size: 24
   Base: 192.168.158.0/24, Size: 24
   Base: 192.168.163.0/24, Size: 24
   Base: 192.168.164.0/24, Size: 24
   Base: 192.168.165.0/24, Size: 24
   Base: 192.168.166.0/24, Size: 24
   Base: 192.168.167.0/24, Size: 24
   Base: 192.168.171.0/24, Size: 24
   Base: 192.168.172.0/24, Size: 24
   Base: 192.168.181.0/24, Size: 24
   Base: 192.168.183.0/24, Size: 24
   Base: 192.168.186.0/24, Size: 24
   Base: 192.168.207.0/24, Size: 24
   Base: 192.168.214.0/24, Size: 24
   Base: 192.168.215.0/24, Size: 24
   Base: 192.168.216.0/24, Size: 24
   Base: 192.168.223.0/24, Size: 24
   Base: 192.168.227.0/24, Size: 24
   Base: 192.168.228.0/24, Size: 24
   Base: 192.168.229.0/24, Size: 24
   Base: 192.168.237.0/24, Size: 24
   Base: 192.168.239.0/24, Size: 24
   Base: 192.168.242.0/24, Size: 24
   Base: 192.168.247.0/24, Size: 24
   Base: fd07:b51a:cc66:d000::/56, Size: 64

WARNING: DOCKER_INSECURE_NO_IPTABLES_RAW is set

# Ubuntu 컨테이너 진입 테스트
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -it --name my-ubuntu ubuntu bash
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -it --name my-ubuntu ubuntu bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
817807f3c64e: Pull complete 
Digest: sha256:186072bba1b2f436cbb91ef2567abca677337cfc786c86e107d25b7072feef0c
Status: Downloaded newer image for ubuntu:latest
root@ea73df58f536:/# ls -F
bin@  boot/  dev/  etc/  home/  lib@  lib64@  media/  mnt/  opt/  proc/  root/  run/  sbin@  srv/  sys/  tmp/  usr/  var/
root@ea73df58f536:/# exit

# Docker hello-world 실행
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:452a468a4bf985040037cb6d5392410206e47db9bf5b7278d281f94d1c2d0931
Status: Downloaded newer image for hello-world:latest

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

```

### 커스텀 이미지 제작 (Dockerfile)

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

## 컨테이너 네트워크 개념(컨테이너 포트 직접 접속이 안 되는 이유)

컨테이너는 **자체 네트워크 공간(네임스페이스)** 안에 존재한다.
호스트와 네트워크가 분리되어 있어서, 컨테이너 내부 포트는 호스트에서 **보이지 않음**.

> 비유: 컨테이너는 건물 안의 방. 방 안에 전화기(포트)가 있어도,
> 외부에서 직접 방 번호로 전화할 수 없음. 건물 대표번호(호스트 포트)로 연결해줘야 함.

---

### 포트 매핑이 필요한 이유

`-p 8080:8080` = 호스트 8080으로 온 요청을 컨테이너 8080으로 전달

- 외부 접근 허용
- 여러 컨테이너가 같은 내부 포트를 써도 호스트 포트만 다르게 하면 충돌 없음
- 필요한 포트만 열어 보안 유지

```bash
# 빌드 및 8080 포트 매핑 실행
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker build -t my-web:1.0 .
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -d -p 8080:80 --name web-server my-web:1.0

# 접속 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % curl http://localhost:8080

# 상태 검증
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
my-web       1.0       74bcd917f983   6 minutes ago   62.2MB
ubuntu       latest    f794f40ddfff   5 weeks ago     78.1MB
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker ps -a
CONTAINER ID   IMAGE        COMMAND                   CREATED          STATUS                     PORTS                                     NAMES
2db9979e9a85   my-web:1.0   "/docker-entrypoint.…"   6 minutes ago    Up 6 minutes               0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-server
ea73df58f536   ubuntu       "bash"                    11 minutes ago   Exited (0) 9 minutes ago                                             my-ubuntu
yhkwon.net4691@c3r6s7 codyssey-e1-1 % 
```

---

## 💾 Step 3 — 볼륨 영속성 검증 

docker volume 구성을 통한 볼륨 영속성 검증

```bash
# 볼륨 생성 및 연결
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker volume create my-data
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -d --name vol-test -v my-data:/app-data ubuntu sleep infinity
816e3df86d7d1dbf7da333cae6807146c980b2dca5fbb587db6b60de3c8f9ea3

# 데이터 쓰기 및 컨테이너 강제 삭제
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker exec vol-test sh -c "echo 'preserved' > /app-data/log.txt"
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker rm -f vol-test
vol-test

# 새로운 컨테이너에서 데이터 유지 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run --rm -v my-data:/app-data ubuntu cat /app-data/log.txt
preserved

# 새로운 컨테이너에서 데이터 유지 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker ps -a
CONTAINER ID   IMAGE        COMMAND                   CREATED          STATUS                      PORTS                                     NAMES
2db9979e9a85   my-web:1.0   "/docker-entrypoint.…"   10 minutes ago   Up 9 minutes                0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-server
ea73df58f536   ubuntu       "bash"                    15 minutes ago   Exited (0) 12 minutes ago                                             my-ubuntu

# 새로운 컨테이너에서 데이터 유지 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker volume ls
DRIVER    VOLUME NAME
local     my-data
yhkwon.net4691@c3r6s7 codyssey-e1-1 % 
```

---

## 🛠️ Step 4 (Bonus) — Docker Compose & 환경 변수

멀티 서비스를 코드로 관리하고 동적으로 설정을 주입했습니다.

### `docker-compose.yml`

```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "${HOST_PORT}:80"
    networks:
      - app-net
  db:
    image: redis:alpine
    networks:
      - app-net

networks:
  app-net:
    driver: bridge
```

### 실행 및 검증

```bash
# 환경 변수 주입 후 실행
yhkwon.net4691@c3r6s7 codyssey-e1-1 % echo "HOST_PORT=9000" > .env
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker-compose up -d

# 서비스 간 네트워크 통신 확인 (Service Discovery)
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker-compose exec web ping -c 3 db
```

---

## 🔑 Step 5 (Bonus) — Git 설정 및 SSH 연동

GitHub 저장소 연동 및 SSH 연동.

```bash
# Git 사용자 설정
yhkwon.net4691@c3r6s7 codyssey-e1-1 % git config --global user.name "0-hu"
yhkwon.net4691@c3r6s7 codyssey-e1-1 % git config --global user.email "yhkwon.net@gmail.com"

# SSH 키 생성 및 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ssh-keygen -t ed25519 -C "yhkwon.net@gmail.com"
yhkwon.net4691@c3r6s7 codyssey-e1-1 % cat ~/.ssh/id_ed25519.pub

# GitHub 연동 테스트
yhkwon.net4691@c3r6s7 codyssey-e1-1 % ssh -T git@github.com
Hi 0-hu! You've successfully authenticated...
```

---

## 🔧 트러블슈팅

### Case 1 — Port Conflict(다른 프로세스에서 사용중인 포트와 충돌)

| 항목 | 내용 |
|------|------|
| 현상 | `docker run` 시 포트 사용 중 에러 발생 |
| 원인 | 이전 실습에서 실행한 `my-web`가 8080 포트를 점유함 |
| 해결 | `docker rm -f my-web` 명령으로 기존 컨테이너를 정리 후 재실행 |

**원인 진단 순서**
```bash
# 1. 포트 점유 프로세스 확인
lsof -i :8080

# 2. 출력 예시
# COMMAND   PID  USER   FD   TYPE  NODE NAME
# docker   1234  user  ...  IPv4  ...  *:8080 (LISTEN)

# 3. 컨테이너가 원인인 경우 정리 후 재실행
docker rm -f my-web
docker run -d -p 8080:8080 --name my-web nginx
```

`lsof -i :포트번호` 로 해당 포트를 점유 중인 PID와 프로세스명을 확인하고,
Docker 컨테이너가 원인임을 파악한 뒤 `docker rm -f` 로 정리하여 해결.

### Case 2 — `.env` 파일 인식 문제

| 항목 | 내용 |
|------|------|
| 현상 | Compose 실행 시 `${HOST_PORT}`가 비어있다는 경고 발생 |
| 원인 | `.env` 파일이 `docker-compose.yml`과 같은 경로에 있지 않음 |
| 해결 | `ls` 명령으로 파일 위치 확인 후 경로 수정하여 해결 |
