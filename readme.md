## REF YouTube Link

For the full 1 hour course watch on youtube:
https://www.youtube.com/watch?v=6YZvp2GwT0A

## REF GIT Link

https://github.com/devopsjourney1/jenkins-101

# Installation

## Build the Jenkins BlueOcean Docker Image (or pull and use the one I built)

```
docker build -t myjenkins:2.41 -f Dockerfile.2.41 .

docker build -t myjenkins:2.45 -f Dockerfile.2.45 .

docker build -t myjenkins:lts -f Dockerfile.lts .

#IF you are having problems building the image yourself, you can pull from my registry (It is version 2.332.3-1 though, the original from the video)

docker pull devopsjourney1/jenkins-blueocean:2.332.3-1 && docker tag devopsjourney1/jenkins-blueocean:2.332.3-1 myjenkins-blueocean:2.332.3-1
```

## Create the network 'jenkins'

```
docker network create jenkins
```

## Run the Container

### MacOS / Linux

```
docker run --name jenkins-2.41 --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data-2.41:/var/jenkins_home \
  --volume jenkins-docker-certs-2.41:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 \
  myjenkins:2.41


docker run --name jenkins-2.41 --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume ./data/jenkins-2.41:/var/jenkins_home \
  --volume ./certs/jenkins-2.41:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 \
  myjenkins:2.41

docker run --name jenkins-2.45 --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data-2.45:/var/jenkins_home \
  --volume jenkins-docker-certs-2.45:/certs/client:ro \
  --publish 8080:8080 --publish 50000:50000 \
  myjenkins:2.45


### 도커 소켓 공유 (리눅스 용)
--volume /var/run/docker.sock:/var/run/docker.sock \

# 로컬과 볼륨 동기화
docker run --name jenkins-2.45-test --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume ./data/jenkins-2.45/:/var/jenkins_home \
  --volume ./certs/jenkins-2.45/:/certs/client:ro \
  --publish 8082:8080 --publish 50000:50000 \
  myjenkins:2.45


docker run --name jenkins-lts --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --volume jenkins-data-lts:/var/jenkins_home \
  --volume jenkins-docker-certs-lts:/certs/client:ro \
  --publish 8082:8080 --publish 50002:50000 \
  myjenkins:lts
```

### Windows

```
docker run --name jenkins-blueocean --restart=on-failure --detach `
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 `
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
  --volume jenkins-data:/var/jenkins_home `
  --volume jenkins-docker-certs:/certs/client:ro `
  --publish 8080:8080 --publish 50000:50000 `
  myjenkins-blueocean:2.414.2


docker run --name jenkins-test --restart=on-failure --detach `
--network jenkins --env DOCKER_HOST=tcp://docker:2376 `
--env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 `
--volume jenkins-data1:/var/jenkins_home `
--volume jenkins-docker-certs1:/certs/client:ro `
-p 8080:8080 --p 50000:50000 `
myjenkins:lts
```

## Get the Password

```
docker exec jenkins-2.45 cat /var/jenkins_home/secrets/initialAdminPassword
```

## Connect to the Jenkins

```
https://localhost:8080/
```

## restart the Jenkins

```

```

## Installation Reference:

https://www.jenkins.io/doc/book/installing/docker/

## alpine/socat container to forward traffic from Jenkins to Docker Desktop on Host Machine

https://stackoverflow.com/questions/47709208/how-to-find-docker-host-uri-to-be-used-in-jenkins-docker-plugin

```
### 원본
docker run -d --restart=always -p 127.0.0.1:2376:2375 \
--name jenkins-socat --network jenkins \
-v /var/run/docker.sock:/var/run/docker.sock \
alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock


# -v /var/run/docker.sock:/var/run/docker.sock \


docker inspect jenkins-socat | grep IPAddress

docker inspect <container_id> | grep IPAddress


tcp://172.17.0.2:2375
tcp://172.21.0.4:2375
```

## Using my Jenkins Python Agent

```
docker pull devopsjourney1/myjenkinsagents:python
```

docker volume inspect jenkins-data-2.45
docker cp <컨테이너 이름 또는 ID>:<컨테이너 내부의 파일 경로> <호스트 시스템의 대상 경로>

docker cp jenkins-2.45:/var/jenkins_home ./data/jenkins-2.45
docker cp jenkins-2.45:/certs/client ./certs/jenkins-2.45

docker run --privileged --name some-docker -d \
-e DOCKER_TLS_CERTDIR=/certs \
 -v some-docker-certs-ca:/certs/ca \
 -v some-docker-certs-client:/certs/client \
 docker:dind

jenkins/agent:jdk17

dnova13/docker-agent
dnova13/jenkins-agent-docker
