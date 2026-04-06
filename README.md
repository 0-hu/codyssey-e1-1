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

docker 명령(run, info, build 등) 및 컨테이너 관리.

```bash
# Docker 버전 및 상태 점검
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker --version
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker info

# Ubuntu 컨테이너 진입 테스트
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -it --name my-ubuntu ubuntu bash
root@a1b2c3d4e5f6:/# ls -F
root@a1b2c3d4e5f6:/# exit
```

### 커스텀 이미지 제작 (Dockerfile)

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

```bash
# 빌드 및 8080 포트 매핑 실행
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker build -t my-web:1.0 .
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -d -p 8080:80 --name web-server my-web:1.0

# 접속 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % curl http://localhost:8080
```

---

## 💾 Step 3 — 볼륨 영속성 검증 

docker volume 구성을 통한 볼륨 영속성 검증

```bash
# 볼륨 생성 및 연결
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker volume create my-data
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run -d --name vol-test -v my-data:/app-data ubuntu sleep infinity

# 데이터 쓰기 및 컨테이너 강제 삭제
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker exec vol-test sh -c "echo 'preserved' > /app-data/log.txt"
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker rm -f vol-test

# 새로운 컨테이너에서 데이터 유지 확인
yhkwon.net4691@c3r6s7 codyssey-e1-1 % docker run --rm -v my-data:/app-data ubuntu cat /app-data/log.txt
preserved
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
| 원인 | 이전 실습에서 실행한 `web-server`가 8080 포트를 점유함 |
| 해결 | `docker rm -f web-server` 명령으로 기존 컨테이너를 정리 후 재실행 |

### Case 2 — `.env` 파일 인식 문제

| 항목 | 내용 |
|------|------|
| 현상 | Compose 실행 시 `${HOST_PORT}`가 비어있다는 경고 발생 |
| 원인 | `.env` 파일이 `docker-compose.yml`과 같은 경로에 있지 않음 |
| 해결 | `ls` 명령으로 파일 위치 확인 후 경로 수정하여 해결 |
