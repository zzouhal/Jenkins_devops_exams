pipeline{
    environment {
        DOCKER_ID = "zzouhal"
        DOCKER_TAG = "latest"
        NP_DEV = 32000
        NP_QA = 32100
        NP_STAGING =32200 
        NP_PROD = 32222
        DOCKERHUB_PWD = credentials("DOCKERHUB_PWD")
        KUBECONFIG = credentials("config")
        MOVIE_DB_PWD = credentials("MOVIE_DB_PWD")
        CAST_DB_PWD = credentials("CAST_DB_PWD")
    }

    agent any
    stages{

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                    if (docker ps -a | grep -q cast-service); then
                        docker stop cast-service
                        docker rm -f cast-service
                    fi
                    if (docker ps -a | grep -q movie-service); then
                        docker stop movie-service
                        docker rm -f movie-service
                    fi
                    if (docker ps -a | grep -q cast-db); then
                        docker stop cast-db
                        docker rm -f cast-db
                    fi
                    if (docker ps -a | grep -q movie-db); then
                        docker stop movie-db
                        docker rm -f movie-db
                    fi
                    if (docker ps -a | grep -q nginx-service); then
                        docker stop nginx-service
                        docker rm -f nginx-service
                    fi
                    docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG ./cast-service
                    docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker run') {
            steps {
                script {
                    sh '''
                    docker run -d --name cast-db \
                        -v postgres_data_cast:/var/lib/postgresql/data/ \
                        -e POSTGRES_USER=cast_db_user \
                        -e POSTGRES_PASSWORD=cast_db_password \
                        -e POSTGRES_DB=cast_db \
                        postgres:12.1-alpine
                    docker run -d --name movie-db \
                        -v postgres_data_movie:/var/lib/postgresql/data/ \
                        -e POSTGRES_USER=movie_db_user \
                        -e POSTGRES_PASSWORD=movie_db_password \
                        -e POSTGRES_DB=movie_db \
                        postgres:12.1-alpine
                    docker run -d --name cast-service \
                        -p 8002:8000 \
                        -e DATABASE_URI=postgresql://cast_db_user:cast_db_password@cast-db:5432/cast_db \
                        $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    docker run -d --name movie-service \
                        -p 8001:8000 \
                        -e DATABASE_URI=postgresql://movie_db_user:movie_db_password@movie-db:5432/movie_db \
                        -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ \
                        $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker run -d --name nginx-service \
                        -p 80:8080 \
                        -v ./nginx_config.conf:/etc/nginx/conf.d/default.conf \
                        nginx:1.17.6-alpine
                    sleep 10
                    '''
                }
            }
        }

             stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                    curl localhost
                    '''
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKERHUB_PWD
                    docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                    '''
                }
            }

        }

        stage('Deploiement en dev'){

            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd ./helm/cast-service
                helm upgrade --install cast-service . --values=values.yaml --namespace dev
                cd ../movie-service/
                helm upgrade --install movie-service . --values=values.yaml --namespace dev
                cd ../nginx/
                helm upgrade --install nginx . --values=values.yaml --namespace dev --set service.NodePort=${NP_DEV}
                helm upgrade --install movie-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace dev \
                --set global.postgresql.auth.postgresPassword=$MOVIE_DB_PWD \
                --set global.postgresql.auth.username=movie \
                --set global.postgresql.auth.password=$MOVIE_DB_PWD \
                --set global.postgresql.auth.database=moviedb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100
                helm upgrade --install cast-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace dev \
                --set global.postgresql.auth.postgresPassword=$CAST_DB_PWD \
                --set global.postgresql.auth.username=cast \
                --set global.postgresql.auth.password=$CAST_DB_PWD \
                --set global.postgresql.auth.database=castdb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100

                '''
                }
            }

            }

        stage('Deploiement en QA'){

            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd ./helm/cast-service
                helm upgrade --install cast-service . --values=values.yaml --namespace qa
                cd ../movie-service/
                helm upgrade --install movie-service . --values=values.yaml --namespace qa
                cd ../nginx/
                helm upgrade --install nginx . --values=values.yaml --namespace qa --set service.NodePort=${NP_QA}
                helm upgrade --install movie-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace qa \
                --set global.postgresql.auth.postgresPassword=$MOVIE_DB_PWD \
                --set global.postgresql.auth.username=movie \
                --set global.postgresql.auth.password=$MOVIE_DB_PWD \
                --set global.postgresql.auth.database=moviedb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100
                helm upgrade --install cast-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace qa \
                --set global.postgresql.auth.postgresPassword=$CAST_DB_PWD \
                --set global.postgresql.auth.username=cast \
                --set global.postgresql.auth.password=$CAST_DB_PWD \
                --set global.postgresql.auth.database=castdb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100

                '''
                }
            }

            }

        stage('Deploiement en staging'){

            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cd ./helm/cast-service
                helm upgrade --install cast-service . --values=values.yaml --namespace staging
                cd ../movie-service/
                helm upgrade --install movie-service . --values=values.yaml --namespace staging
                cd ../nginx/
                helm upgrade --install nginx . --values=values.yaml --namespace staging --set service.NodePort=${NP_STAGING}
                helm upgrade --install movie-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace staging \
                --set global.postgresql.auth.postgresPassword=$MOVIE_DB_PWD \
                --set global.postgresql.auth.username=movie \
                --set global.postgresql.auth.password=$MOVIE_DB_PWD \
                --set global.postgresql.auth.database=moviedb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100
                helm upgrade --install cast-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                --namespace staging \
                --set global.postgresql.auth.postgresPassword=$CAST_DB_PWD \
                --set global.postgresql.auth.username=cast \
                --set global.postgresql.auth.password=$CAST_DB_PWD \
                --set global.postgresql.auth.database=castdb \
                --set livenessProbe.initialDelaySeconds=100 \
                --set readinessProbe.initialDelaySeconds=100

                '''
                }
            }   

            }


        stage('Deploiement en prod'){

            when {
                branch 'master'
            }
            steps {
            
                timeout(time: 5, unit: "MINUTES") {
                    input message: 'Do you want to deploy to production ?', ok: 'Yes'
                    }

                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cd ./helm/cast-service
                    helm upgrade --install cast-service . --values=values.yaml --namespace prod
                    cd ../movie-service/
                    helm upgrade --install movie-service . --values=values.yaml --namespace prod
                    cd ../nginx/
                    helm upgrade --install nginx . --values=values.yaml --namespace prod --set service.NodePort=${NP_PROD}
                    helm upgrade --install movie-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                    --namespace prod \
                    --set global.postgresql.auth.postgresPassword=$MOVIE_DB_PWD \
                    --set global.postgresql.auth.username=movie \
                    --set global.postgresql.auth.password=$MOVIE_DB_PWD \
                    --set global.postgresql.auth.database=moviedb \
                    --set livenessProbe.initialDelaySeconds=100 \
                    --set readinessProbe.initialDelaySeconds=100
                    helm upgrade --install cast-db oci://registry-1.docker.io/bitnamicharts/postgresql \
                    --namespace prod \
                    --set global.postgresql.auth.postgresPassword=$CAST_DB_PWD \
                    --set global.postgresql.auth.username=cast \
                    --set global.postgresql.auth.password=$CAST_DB_PWD \
                    --set global.postgresql.auth.database=castdb \
                    --set livenessProbe.initialDelaySeconds=100 \
                    --set readinessProbe.initialDelaySeconds=100

                '''
                }
                }
            }


    }
}