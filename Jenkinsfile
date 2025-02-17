pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'my-nginx:latest'
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Build docker image ..."
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        
        stage('Scan Image with Trivy') {
            steps {
                script {
                    def trivyStatus = sh(script: "trivy image --severity HIGH,CRITICAL --exit-code 1 --no-progress ${DOCKER_IMAGE}", returnStatus: true)
                    if (trivyStatus == 0) {
                        echo "Trivy scan passed - no HIGH or CRITICAL vulnerabilities found"
                    } else {
                        error "Trivy scan failed - HIGH or CRITICAL vulnerabilities found"
                    }
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    docker.image(DOCKER_IMAGE).run('-p 8080:80 --name my-nginx-container')
                    curl http://localhost:8080/
                }
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed. The image may have vulnerabilities or there was an error.'
        }
        success {
            echo 'Pipeline succeeded. The container is now running.'
        }
    }
}
