pipeline {
    agent {
        label 'agent-in-docker'
    }
    // 젠킨스 파이프라인에서 트리거를 적용하기 위해서는 반드시 설정
    // triggers {
    //     // 5분 마다 git 에 새로운 코드가 있으면 빌드를 실행함.
    //     pollSCM '*/5 * * * *'
    //     // pollSCM '*/5 * * * *'
    // }
    environment {
        DB_HOST = credentials('DB_HOST')
        DB_NAME = credentials('DB_NAME')
        DB_USER = credentials('DB_USER')
        DB_PASS = credentials('DB_PASS')
        DB_PORT = credentials('DB_PORT')
        TIME_ZONE = credentials('TIME_ZONE')
        DOCKER_ID = credentials('DOCKER_ID')
        DOCKER_ACC_KEY = credentials('DOCKER_ACC_KEY')
        
        JENKINS_PROJECT="${env.JOB_NAME}"
        PROJECT_NAME="node-server-sample"

    }
    stages {
        stage('Git Clone') {
            steps {
                git branch: 'master',
                url: 'https://github.com/dnova13/node-server-sample.git', 
                // 레포지토리가 private 일경우, credentials 에 설정한 git 계정 정보를 설정
                credentialsId: 'git-signin'

            }
            
            post {
                success { 
                    sh '##################### echo "Successfully Cloned Repository"'
                }
                failure {
                    sh '##################### echo "Fail Cloned Repository"'
                }
            }    
        }

        stage('Generate .env file') {
            steps {

                sh '''
                echo "DB_HOST=$DB_HOST" >> .env
                echo "DB_NAME=$DB_NAME" >> .env
                echo "DB_USER=$DB_USER" >> .env
                echo "DB_PASS=$DB_PASS" >> .env
                echo "DB_PORT=$DB_PORT" >> .env
                echo "DB_POOL=20" >> .env
                echo "TIME_ZONE=$TIME_ZONE" >> .env
                '''
                sh 'cat .env '
            }
        }

        stage('Build') {
            steps {
                sh '''                
                docker-compose up --build -d mariadb
                sleep 30
                docker cp ./mariadb/sqls/ mariadb:/docker-entrypoint-initdb.d/
                docker exec -i mariadb mariadb -u $DB_USER -p$DB_PASS video-platform < ./mariadb/sqls/video-platform_dump20240623_main_back.sql
                docker-compose up --build -d backend

                '''
            }
            post {
                success {
                    echo '##################### docker build success'
                }

                failure {
                    echo '##################### docker build failed'
                }
            }
        }
        
        stage('Build Test') {
            steps {
            	// gralew이 있어야됨. git clone해서 project를 가져옴.
                sh 'docker exec backend npm run api-test'
            }
            post {
                success {
                    echo '##################### build test success'
                }

                failure {
                    echo '##################### build test failed'
                }
            }
        }
        stage('Build Docker Push') {
            steps {
                sh 'docker login -u $DOCKER_ID -p $DOCKER_ACC_KEY'
                sh '''

                echo "The project name is $PROJECT_NAME"

                docker images
                docker-compose build webserver
                docker build -t $DOCKER_ID/$PROJECT_NAME-webserver-backup ./nginx_webserver

                # docker tag
                docker tag $JENKINS_PROJECT-backend $DOCKER_ID/$PROJECT_NAME-backend:latest
                docker tag $JENKINS_PROJECT-webserver $DOCKER_ID/$PROJECT_NAME-webserver:latest
                docker tag $JENKINS_PROJECT-mariadb $DOCKER_ID/$PROJECT_NAME-mariadb:latest

                # docker push
                docker push $DOCKER_ID/$PROJECT_NAME-backend
                docker push $DOCKER_ID/$PROJECT_NAME-webserver
                docker push $DOCKER_ID/$PROJECT_NAME-webserver-backup
                docker push $DOCKER_ID/$PROJECT_NAME-mariadb

                '''
            }
            post {
                success {
                    echo '##################### docker push success'
                }

                failure {
                    echo '##################### docker push failed'
                }
            }
        }
        stage('SSH Deploy') {
            steps {        
                sshagent (credentials: ['ec3-test']) {
                sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@3.39.6.193 '
                        # 도커 허브 로그인
                        docker login -u $DOCKER_ID -p $DOCKER_ACC_KEY

                        mkdir ~/project/$PROJECT_NAME

                        # Docker Compose로 애플리케이션 빌드 및 실행
                        cd ~/project/$PROJECT_NAME

                        sudo docker pull $DOCKER_ID/$PROJECT_NAME-backend:latest
                        sudo docker pull $DOCKER_ID/$PROJECT_NAME-webserver:latest
                        sudo docker pull $DOCKER_ID/$PROJECT_NAME-webserver-backup:latest
                        sudo docker pull $DOCKER_ID/$PROJECT_NAME-mariadb:latest

                        # 도커 이미지 조회
                        sudo docker images  
                        sudo docker network create platform-network

                        # backend-backup 실행
                        sudo docker run -d \
                            --name backend-backup \
                            --memory=256m \
                            --restart=on-failure \
                            -v ./uploads:/app/project/uploads:ro \
                            -v ./.env:/app/project/.env:ro \
                            --network=platform-network \
                            $DOCKER_ID/$PROJECT_NAME-backend
                            

                        # 기존 webserver 중단
                        sudo docker rm -f webserver

                        # webserver-backup 실행  
                        sudo docker run -d \
                            --name webserver-backup \
                            --memory=256m \
                            --restart=always \
                            -v ./nginx_webserver/default_backup.conf:/etc/nginx/conf.d/default.conf \
                            -p 80:80 \
                            -p 3010:3010 \
                            --network=platform-network \
                            $DOCKER_ID/$PROJECT_NAME-webserver-backup

                        # mariadb 실행
                            sudo docker run -d \
                            --name mariadb \
                            --memory=128m \
                            --restart=unless-stopped \
                            -v ./mariadb/maria_data:/var/lib/mysql \
                            -v ./mariadb/sqls/:/docker-entrypoint-initdb.d/ \
                            -e MARIADB_DATABASE=$DB_NAME \
                            -e MARIADB_USER=$DB_USER \
                            -e MARIADB_ROOT_PASSWORD=$DB_PASS \
                            -e TZ=$TIME_ZONE \
                            --expose=3306 \
                            --network=platform-network \
                            $DOCKER_ID/$PROJECT_NAME-mariadb


                        # 실 백엔드 서버 리빌드 
                        echo "--------------------- backend rebuild ---------------------------" 
                        sudo docker rm -f backend
                        sudo docker run -d \
                            --name backend \
                            --memory=256m \
                            --restart=on-failure \
                            -v ./uploads:/app/project/uploads \
                            -v ./.env:/app/project/.env \
                            --network=platform-network \
                            $DOCKER_ID/$PROJECT_NAME-backend

                        # 백업 컨테이너 중단
                        sudo docker rm -f backend-backup
                        sudo docker rm -f webserver-backup   

                        # 기존 webserver 재실행
                        sudo docker run -d \
                            --name webserver \
                            --memory=256m \
                            --restart=always \
                            -v ./nginx_webserver/default.conf:/etc/nginx/conf.d/default.conf \
                            -v ./nginx_webserver/logs:/var/log/nginx \
                            -p 80:80 \
                            -p 3010:3010 \
                            --network=platform-network \
                            $DOCKER_ID/$PROJECT_NAME-webserver


                        # 도커 캐시 제거
                        sudo docker system prune -af
                    '
                """
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh 'docker-compose down -v --rmi all'
            }
        }
    }
}