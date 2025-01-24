pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "nguray" // replace this with your docker-id
DOCKER_IMAGE = "examenapi"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/"${DOCKER_IMAGE}_movie":$DOCKER_TAG ./movie-service
                 docker build -t $DOCKER_ID/"${DOCKER_IMAGE}_cast":$DOCKER_TAG ./cast-service
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
            steps {
                script {
                sh '''


                docker network create -d bridge my-net

                docker run -d  -it --rm --network=my-net \
                --name moviedb \
                -e POSTGRES_PASSWORD=movie_db_password \
                -e POSTGRES_USER=movie_db_username \
                -e POSTGRES_DB=movie_db_dev \
                -v postgres_data_movie:/var/lib/postgresql/data/ \
                postgres:12.1-alpine
                sleep 5
                docker run -d  -it --rm --network=my-net \
                --name castdb \
                -e POSTGRES_PASSWORD=cast_db_password \
                -e POSTGRES_USER=cast_db_username \
                -e POSTGRES_DB=cast_db_dev \
                -v postgres_data_cast:/var/lib/postgresql/data \
                postgres:12.1-alpine
                sleep 5
                docker run -it -d --rm --network=my-net --name movie $DOCKER_ID/"${DOCKER_IMAGE}_movie":$DOCKER_TAG sh -c "uvicorn app.main:app --reload --host 0.0.0.0 --port 8000"
                sleep 5
                docker run -it -d --rm --network=my-net --name cast $DOCKER_ID/"${DOCKER_IMAGE}_cast":$DOCKER_TAG sh -c "uvicorn app.main:app --reload --host 0.0.0.0 --port 8000"
                sleep 5
                docker run -d  -it --rm --network=my-net \
                --name app \
                -p 8081:8080 \
                -v ./nginx_config.conf:/etc/nginx/conf.d/default.conf \
                nginx:latest

                '''
                }
            }
        }
        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''

                    curl http://localhost:8081/api/v1/movies/docs
                    curl http://localhost:8081/api/v1/casts/docs

                    docker stop app movie cast castdb moviedb

                    docker network rm my-net

                    sleep 5
                    '''
                    }
            }

        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/"${DOCKER_IMAGE}_movie":$DOCKER_TAG
                docker push $DOCKER_ID/"${DOCKER_IMAGE}_cast":$DOCKER_TAG
                '''
                }
            }

        }

        stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                kubectl apply -f ps-cast-deploy.yaml --namespace dev
                sleep 5
                kubectl apply -f ps-movie-deploy.yaml --namespace dev
                sleep 5
                cp casts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appcasts casts --values=values.yml --namespace dev
                sleep 5
                cp movies/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appmovies movies --values=values.yml --namespace dev
                sleep 5
                kubectl apply -f nginx-deployment.yaml --namespace dev
                '''
                }
            }

        }

        stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp casts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appcasts casts --values=values.yml --namespace qa
                sleep 5
                cp movies/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appmovies movies --values=values.yml --namespace qa
                sleep 5
                kubectl apply -f nginx-deployment.yaml --namespace qa

                '''
                }
            }

        }

        stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp casts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appcasts casts --values=values.yml --namespace staging
                sleep 5
                cp movies/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appmovies movies --values=values.yml --namespace staging
                sleep 5
                kubectl apply -f nginx-deployment.yaml --namespace staging

                '''
                }
            }

        }


        stage('Deploiement en prod'){
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            when {
                expression {env.GIT_BRANCH == 'origin/master' | env.GIT_BRANCH == 'origin/main'}
            }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp casts/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appcasts casts --values=values.yml --namespace prod
                sleep 5
                cp movies/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install appmovies movies --values=values.yml --namespace prod
                sleep 5
                kubectl apply -f nginx-deployment.yaml --namespace prod

                '''
                }
            }

        }




}
}


