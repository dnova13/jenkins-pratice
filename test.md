```
pipeline {
    // agent {
    //     // Docker Agent 설정
    //     docker {
    //         image 'docker:latest'
    //         args '-v /var/run/docker.sock:/var/run/docker.sock'
    //     }
    // }
    // environment {
    //     DOCKER_BUILDKIT = '1'
    // }
    agent {
        node {
            // label 'docker-agent-docker' // 이 에이전트를 사용할 노드의 레이블
            label 'docker-agent-alpine' // 이 에이전트를 사용할 노드의 레이블
        }
    }
    stages {
         stage('Docker Build and Test') {
            steps {
                script {
                    sh '''
                    echo "afsdfsdfsfdfsdfsdfsdsf"
                    docker version
                    '''
                    sh 'docker ps'
                    sh 'docker compose up -d --build'
                    sleep 30
                    sh 'docker exec backend npm run api-test'
                }
            }
        }
    }
}
```

docker exec -it jenkins-2.45 bash

docker run --name jenkins-2.45 --restart=on-failure --detach \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume jenkins-data-2.45:/var/jenkins_home \
 --volume jenkins-docker-certs-2.45:/certs/client:ro \
 --publish 8081:8080 --publish 50000:50000 \
 myjenkins:2.45

###

docker run --name jenkins-2.45 --restart=on-failure --detach \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume jenkins-data-2.45:/var/jenkins_home \
 --volume jenkins-docker-certs-2.45:/certs/client:ro \
 --publish 8081:8080 --publish 50000:50000 \
 myjenkins:2.45

###

jenkins/agent:latest-jdk17

### 도커 생성

docker build -t dnova13/jenkins-agent-docker -f Dockerfile.docker .
docker push dnova13/jenkins-agent-docker

docker run \
 -p 8080:8080 \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume jenkins-data-2.45:/var/jenkins_home \
 --volume jenkins-docker-certs-2.45:/certs/client:ro \
 -v /var/run/docker.sock:/var/run/docker.sock \
 --name jenkins \
 getintodevops/jenkins-withdocker:lts

docker run --name jenkins-2.45 --restart=on-failure --detach \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume jenkins-data-2.45:/var/jenkins_home \
 --volume jenkins-docker-certs-2.45:/certs/client:ro \
 -v /var/run/docker.sock:/var/run/docker.sock \
 --publish 8081:8080 --publish 50000:50000 \
 myjenkins:2.45

--volume /var/run/docker.sock:/var/run/docker.sock \

docker run --name jenkins-2.45 --restart=on-failure --detach \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume jenkins-data-2.45:/var/jenkins_home \
 --volume jenkins-docker-certs-2.45:/certs/client:ro \
--publish 8081:8080 --publish 50000:50000 \
 myjenkins:2.45

### 도커 소켓 공유 (리눅스 용)

docker run --name jenkins-2.45 --restart=on-failure --detach \
-v /var/run/docker.sock:/var/run/docker.sock \
--publish 8080:8080 --publish 50000:50000 \
 myjenkins:2.45

docker run --name jenkins --restart=on-failure --detach \
-v /var/run/docker.sock:/var/run/docker.sock \
--volume jenkins-data:/var/jenkins_home \
--volume jenkins-certs:/certs/client:ro \
--publish 8080:8080 --publish 50000:50000 \
 jenkins/jenkins:lts-jdk17

docker build -t myjenkins -f Dockerfile.docker .

docker run --rm -d -u root -p 8080:8080 -v jenkins-data:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock -v "$HOME":/home myjenkins

-v /var/run/docker.sock:/var/run/docker.sock \

#### 되는거 -u root 붙이니 됐다

docker run --name jenkins --restart=on-failure -d -u root \
--volume /var/run/docker.sock:/var/run/docker.sock \
--volume jenkins-data1:/var/jenkins_home \
--volume jenkins-certs1:/certs/client:ro \
--publish 8080:8080 --publish 50000:50000 \
 myjenkins

## --volume /var/run/docker.sock:/var/run/docker.sock \

docker run --name jenkins --restart=on-failure -d \
--volume /var/run/docker.sock:/var/run/docker.sock \
--volume jenkins-data2:/var/jenkins_home \
--volume jenkins-certs2:/certs/client:ro \
--publish 8080:8080 --publish 50000:50000 \
 myjenkins

docker run --name jenkins --restart=on-failure --detach \
-v /var/run/docker.sock:/var/run/docker.sock \
--volume jenkins-data:/var/jenkins_home \
--volume jenkins-certs:/certs/client:ro \
--publish 8080:8080 --publish 50000:50000 \
 jenkins/jenkins:lts-jdk17

## 로컬 볼륨 동기화

docker run --name jenkins-2.45-test --restart=on-failure --detach -u root \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume ./data/jenkins-2.45/:/var/jenkins_home \
 --volume ./certs/jenkins-2.45/:/certs/client:ro \
 --publish 8082:8080 --publish 50001:50000 \
 myjenkins:2.45

# test jenkins

https://github.com/darinpope/jenkins-example-docker/blob/main/Jenkinsfile-1

---

---

---

---

---

---

---

---

---

---

---

---

---

---

---

---

# 젠킨스 설정 - 축약설정

docker build -f Dockerfile.docker -t jenkins_docker .

```
docker run --name jenkins-docker --restart=on-failure --detach -u root \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume jenkins-docker-data:/var/jenkins_home \
 --volume jenkins-docker-cert:/certs/client:ro \
 --publish 8081:8080 --publish 50002:50000 \
--network jenkins \
 jenkins_docker

 #
 # --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 # --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
```

###

###

# 젠킨스 공식 문서 설정

docker build -f Dockerfile.lts -t jenkins_lts .

```

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

#

#

# docke 재설계

docker build -t jenkins.2.45.1 -f Dockerfile.2.45.1 .

<!-- docker run --name test1 --restart=on-failure --detach -u root \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume ./jenkins-2.45.1/data/:/var/jenkins_home \
 --volume ./jenkins-2.45.1/certs/:/certs/client:ro \
 --publish 8080:8080 --publish 50000:50000 \
 jenkins.2.45.1 -->

--env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \

docker run --name test1 --restart=on-failure --detach -u root \
 --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
 --volume /var/run/docker.sock:/var/run/docker.sock \
 --volume jenkins-2.45.1-data1:/var/jenkins_home \
 --volume jenkins-2.45.1-certs1:/certs/client:ro \
 --publish 8080:8080 --publish 50000:50000 \
 jenkins.2.45.1

docker build -t jenkins.2.45.1 -f Dockerfile.2.45.1 .

# 도커 설치된 어젠트 설치

docker build -t dnova13/docker-agent -f Dockerfile.agent-docker .
docker build -t dnova13/docker-agent1 -f Dockerfile.agent-docker1 .

docker push dnova13/docker-agent1

```

```

### jenkins-slave build

docker build -t dnova13/jenkins-slave -f Dockerfile.jenkins-slave .

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


docker run -d --restart=always -p 2376:2375 \
--name jenkins-socat --network jenkins \
-v /var/run/docker.sock:/var/run/docker.sock \
alpine/socat tcp-listen:2375,fork,reuseaddr unix-connect:/var/run/docker.sock

##
tcp://172.17.0.2:2375

socat 없이 로컬 도커로 연결
unix:///var/run/docker.sock
```

## docker in docker (dind) test

```
docker build -t dnova13/docker-agent -f Dockerfile.agent-docker .

# 접속 테스트
docker run -e DOCKER_HOST=unix:///var/run/docker.sock -it  dnova13/docker-agent bash

/home/jenkins/agent

```
