# dev-workspace

개발 워크스테이션 구축 과제 1 — WSL2 + Ubuntu + Docker 개발환경 구축

## 1. 환경 정보

| 항목 | 버전 |
|------|------|
| WSL | 3.0.3.0 (WSL2) |
| Ubuntu | 26.04.1 LTS |
| Docker | 29.9.2 (build a82e528) |

## 2. 저장소 구조

```
dev-workspace/
├── README.md
├── get-docker.sh        # Docker 공식 설치 스크립트
├── docker-env/
│   └── Dockerfile       # ubuntu:latest 기반 개발용 이미지
└── images/              # 실행 결과 스크린샷
```

## 3. 설치 및 실행 과정

### 3-1. Docker 설치 확인

```bash
docker --version
```

![docker version](images/docker-version.png)

### 3-2. hello-world 컨테이너 실행

```bash
docker run hello-world
```

Docker 클라이언트가 데몬에 요청 → Docker Hub에서 이미지 pull → 컨테이너 생성 및 실행 → 출력 전달 순서로 동작함을 확인했다.

![hello-world](images/hello-world.png)

### 3-3. ubuntu 컨테이너 실행

```bash
docker run -it ubuntu bash
cat /etc/os-release
exit
```

`-i`(입력 유지)와 `-t`(터미널 할당)로 컨테이너 안의 bash에 접속했다. 컨테이너 내부 OS가 Ubuntu 26.04.1 LTS임을 확인했다.

![ubuntu run](images/ubuntu-run.png)

### 3-4. 컨테이너 목록 확인

```bash
docker ps -a
```

`-a` 옵션은 종료된(Exited) 컨테이너까지 모두 보여준다. hello-world, ubuntu, my-ubuntu 컨테이너가 실행 후 정상 종료(Exited (0))된 것을 확인할 수 있다.

![docker ps](images/docker-ps.png)

### 3-5. 이미지 목록 확인

```bash
docker images
```

![docker images](images/docker-images.png)

### 3-6. 파일 권한 설정

```bash
chmod 755 get-docker.sh
ls -l
```

`755` = 소유자 읽기/쓰기/실행(rwx), 그룹·기타 읽기/실행(r-x). 실행 후 `get-docker.sh`의 권한이 `-rwxr-xr-x`로 바뀐 것을 확인했다.

![chmod](images/chmod-ls.png)

### 3-7. Dockerfile 작성

`docker-env/Dockerfile`

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    git \
    curl

WORKDIR /app

CMD ["/bin/bash"]
```

### 3-8. 이미지 빌드

```bash
docker build -t my-ubuntu ./docker-env
```

![docker build](images/docker-build.png)

### 3-9. 포트 매핑 및 접속 테스트

`my-ubuntu`에는 웹 서버가 없으므로 nginx 이미지로 테스트한다.

```bash
docker run -d -p 8080:80 --name web nginx
curl http://localhost:8080
```

`-d`는 백그라운드 실행, `-p 8080:80`은 호스트 8080 포트를 컨테이너 80 포트에 연결한다. 로컬에 nginx 이미지가 없어 Docker Hub에서 자동으로 pull한 뒤 실행되었다.

![port mapping](images/port-mapping.png)

호스트에서 `localhost:8080`으로 요청하자 컨테이너의 nginx가 응답(`Welcome to nginx!`)하여 포트 매핑이 정상 동작함을 확인했다.

![curl test](images/curl-test.png)

## 4. 개념 정리

### WSL2
Windows Subsystem for Linux 2. Windows 안에서 경량 가상머신으로 실제 Linux 커널을 실행해, 별도 VM 설치 없이 Linux 환경을 쓸 수 있게 해준다. WSL1보다 파일 시스템 성능과 시스템 호출 호환성이 좋아 Docker 실행이 가능하다.

### Docker
애플리케이션과 실행에 필요한 라이브러리·설정을 하나의 이미지로 묶어, 어느 환경에서든 같은 방식으로 실행할 수 있게 해주는 컨테이너 플랫폼이다. "내 PC에선 되는데" 문제를 줄여준다.

### 컨테이너 vs 가상머신

| 구분 | 컨테이너 | 가상머신 |
|------|----------|----------|
| 격리 단위 | 프로세스 (호스트 커널 공유) | OS 전체 (게스트 OS 포함) |
| 크기 | MB 단위 | GB 단위 |
| 시작 시간 | 초 단위 | 분 단위 |
| 자원 사용 | 가벼움 | 무거움 |
| 격리 수준 | 상대적으로 약함 | 강함 |

### 이미지 vs 컨테이너
- **이미지**: 실행에 필요한 파일과 설정을 담은 읽기 전용 템플릿 (클래스에 비유)
- **컨테이너**: 이미지를 실행한 인스턴스. 하나의 이미지로 여러 컨테이너를 만들 수 있다 (객체에 비유)

### Dockerfile 주요 문법

| 명령 | 의미 |
|------|------|
| `FROM` | 기반 이미지 지정 |
| `RUN` | 빌드 시 실행할 명령 (패키지 설치 등) |
| `WORKDIR` | 이후 명령의 작업 디렉터리 |
| `COPY` | 호스트 파일을 이미지로 복사 |
| `ENV` | 환경 변수 설정 |
| `EXPOSE` | 컨테이너가 사용하는 포트 명시 |
| `CMD` | 컨테이너 시작 시 기본 실행 명령 |

### 포트 매핑
`-p 호스트포트:컨테이너포트` 형식. 예) `-p 8080:80`은 호스트의 8080번 포트로 들어온 요청을 컨테이너의 80번 포트로 전달한다. 컨테이너는 격리된 네트워크를 쓰기 때문에 외부에서 접속하려면 포트 매핑이 필요하다.

### 볼륨 마운트
`-v 호스트경로:컨테이너경로` 형식. 컨테이너는 삭제되면 내부 데이터도 사라지므로, 호스트 디렉터리를 연결해 데이터를 유지하거나 코드를 실시간으로 공유한다.

```bash
docker run -it -v $(pwd):/app my-ubuntu
```

### 환경 변수
`-e KEY=VALUE` 옵션이나 Dockerfile의 `ENV`로 설정한다. 비밀번호·설정값을 이미지에 하드코딩하지 않고 실행 시점에 주입할 수 있다.

```bash
docker run -e APP_ENV=dev my-ubuntu env
```
