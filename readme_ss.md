# 젠킨스 설정 - 간략화 (ssl X)

docker build -f Dockerfile.lts -t jenkins_docker .

```
docker run --name jenkins-docker --restart=on-failure --detach -u root \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume jenkins-docker-data:/var/jenkins_home \
 --volume jenkins-docker-cert:/certs/client:ro \
 --publish 8081:8080 --publish 50002:50000 \
--network jenkins \
 jenkins_docker

 # ssl 설정 제외 시킴
 # --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 # --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
```

# 젠킨스 공식 문서 설정

```
# lts 빌드
docker build -f Dockerfile.lts -t jenkins_lts .
```

```
# jenkins 컨테이너 생성
docker run --name jenkins-lts --restart=on-failure --detach -u root \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume ./jenkins-lts/data/:/var/jenkins_home \
 --volume ./jenkins-lts/certs/:/certs/client:ro \
 --publish 8080:8080 --publish 50000:50000 \
 --network jenkins \
 --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 jenkins_lts


## 이거 ssl, tls 설정 등에 필요
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 # privileged
```

# 도커 설치된 어젠트 설치

```
docker build -t dnova13/docker-agent -f Dockerfile.agent-docker .

docker push dnova13/docker-agent
```

## socat test

```
docker run --privileged --name some-docker -d \
-e DOCKER_TLS_CERTDIR=/certs \
 -v some-docker-certs-ca:/certs/ca \
 -v some-docker-certs-client:/certs/client \
 docker:dind


### 직접 연결하지 않고 중계기로 연결
docker run -d --restart=always -p 127.0.0.1:2376:2375 \
--name jenkins-socat --network jenkins \
-v /var/run/docker.sock:/var/run/docker.sock \
alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock

## 어젠트 외부 연결용
docker run -d --restart=always -p 2376:2375 \
--name jenkins-socat --network jenkins \
-v /var/run/docker.sock:/var/run/docker.sock \
alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock


### docker socat container ip 검색
docker inspect jenkins-socat | grep IPAddress

### socat container ip 검색 적용 예시
tcp://172.17.0.2:2375

### socat 사용안한 로컬 도커로 연결
unix:///var/run/docker.sock

### vm 머신 연결
tcp://192.168.56.1:2375
```

## docker in docker (dind) test

```
docker build -t dnova13/docker-agent -f Dockerfile.agent-docker .

# 접속 테스트
docker run -e DOCKER_HOST=unix:///var/run/docker.sock -it  dnova13/docker-agent bash


# 이게 되는거
docker run -it -u root \
-v /var/run/docker.sock:/var/run/docker.sock  \
--network jenkins \
 dnova13/docker-agent bash


# 아래부터 실패한 코드 및 참조용으로 정리

docker run -it \
-v /var/run/docker.sock:/var/run/docker.sock  \
--network jenkins \
-e DOCKER_HOST=unix:///var/run/docker.sock \
 dnova13/docker-agent bash


docker run -it -u root \
-e DOCKER_HOST=unix:///var/run/docker.sock \
--network jenkins \
 dnova13/docker-agent bash


docker run -it -u root \
--network jenkins \
 dnova13/docker-agent bash

docker run -it -u root \
-e DOCKER_HOST=tcp://0.0.0.0:2376 \
 dnova13/docker-agent bash

docker run -it -u root \
-v /var/run/docker.sock:/var/run/docker.sock  \
 dnova13/docker-agent bash

-e DOCKER_HOST=tcp://docker:2376

docker run -e DOCKER_HOST=unix:///var/run/docker.sock  -v /var/run/docker.sock:/var/run/docker.sock -it dnova13/docker-agent bash
```
