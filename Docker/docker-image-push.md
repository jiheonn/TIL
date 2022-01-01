- [Docker 이미지 배포하기](#docker-이미지-배포하기)
  - [Docker Hub](#docker-hub)

# Docker 이미지 배포하기

## Docker Hub

[Docker Hub](https://hub.docker.com/)에서 회원가입 후 로그인을 진행한다.

Repositories로 이동한 후 Create Repository를 클릭한다.

<img width="1392" alt="" src="https://user-images.githubusercontent.com/48443734/147847257-6625c05c-22be-4fdb-bd73-b8bd4c437892.png" />

생성할 Repository의 이름을 입력한 후 공개 여부를 선택하고 Create를 클릭한다. Private은 계정당 무료로 하나만을 제공하기 때문에 더 필요한 경우 비용을 지불해야 한다.

<img width="1392" alt="" src="https://user-images.githubusercontent.com/48443734/147847270-d2334b15-bd07-42c3-9c99-7ab0b341f5e4.png" />

생성된 Repository를 확인한 후 사전 준비를 마친다.

<img width="1392" alt="" src="https://user-images.githubusercontent.com/48443734/147847402-27c83624-ca9c-49be-b3c3-fc4446909732.png" />

CLI 환경에서 Docker Hub로 이미지를 업로드하기 위해 먼저 로그인을 진행한다.

```bash
$ docker login
```

`docker push`를 통해 원하는 이미지를 업로드한다.

```bash
$ docker push <username>/<image>:<tag>

# ex) docker push jiheon/apache-server:1.0
```

Docker Hub에 이미지가 업로드된 것을 확인할 수 있다.

<img width="1392" alt="" src="https://user-images.githubusercontent.com/48443734/147847693-4e1d334f-2dce-4a6d-9f4f-f9b0b4087d7b.png">

`docker pull`을 통해 Docker Hub에 업로드된 이미지를 다운로드할 수 있다.

```bash
$ docker pull <username>/<image>:<tag>

# ex) docker pull jiheon/apache-server:1.0
```