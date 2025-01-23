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
                -name moviedb \
                -e POSTGRES_PASSWORD=movie_db_password \
                -e POSTGRES_USER=movie_db_username \
                -e POSTGRES_DB=movie_db_dev \
                -v postgres_data_movie:/var/lib posgresql/data/ postgres:12.1-alpine
                '''
                }
            }
        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    sleep 10
                    '''
                    }
            }

        }


}
}


