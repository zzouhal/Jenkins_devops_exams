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

        // stage('run docker compose'){
        //     steps{
        //         script{
        //         sh '''
        //           docker-compose up -d
        //         '''
        //         }
        //     }
        // }

        // stage('test docker compose') {
        //     steps{
        //         script{
        //         sh '''
        //         curl -I GET http://localhost:8080/api/v1/movies/docs 
        //         curl -I  GET http://localhost:8080/api/v1/casts/docs
        //         '''
        //         }
        //     }
        // }

        // stage('stop docker compose') {
        //     steps{
        //         script{
        //         sh '''
        //         docker-compose down
        //         '''
        //         }
        //     }
        // }

        stage('Build movie service'){
            environment
            {
                DOCKER_IMAGE = "movie-service"
            }

            steps {
                script {
                sh '''
                cd $DOCKER_IMAGE/
                docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                sleep 6
                '''
                }
            }

        }

        stage('Build cast service'){
            environment
            {
                DOCKER_IMAGE = "cast-service"
            }

            steps {
                script {
                sh '''
                cd $DOCKER_IMAGE/
                docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                sleep 6
                '''
                }
            }

        }

        stage('Docker run cast service'){
            environment
            {
                DOCKER_IMAGE = "cast-service"
            }
                steps {
                    script {
                    sh ''' 
                    docker run -d -p 8002:8000 --name cast-service $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    curl localhost:8002
                    '''
                    }
                }
            }


            stage('Docker run movie service'){
            environment
            {
                DOCKER_IMAGE = "movie-service"
            }
                steps {
                    script {
                    sh '''
                    docker run -d -p 8001:8000 --name movie-service $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    sleep 10
                    curl localhost:8001
                    '''
                    }
                }
            }

        // stage('Test Acceptance movie service'){
        //     steps {
        //             script {
        //             sh '''
        //             curl localhost:8000
        //             '''
        //             }
        //     }
        // }

        //  stage('Test Acceptance cast service'){
        //     steps {
        //             script {
        //             sh '''
        //             curl localhost:8002
        //             '''
        //             }
        //     }
        // }

        stage('Push cast service'){
            environment
            {
                DOCKER_IMAGE = "cast-service"
            }

            steps {
                script {
                sh '''
                cd $DOCKER_IMAGE/
                docker login -u $DOCKER_ID -p $DOCKERHUB_PWD
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }

        stage('Push movie service'){
            environment
            {
                DOCKER_IMAGE = "movie-service"
            }

            steps {
                script {
                sh '''
                cd $DOCKER_IMAGE/
                docker login -u $DOCKER_ID -p $DOCKERHUB_PWD
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
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