    environment {
        DOCKER_USERNAME = '{username'
        IMAGE_NAME = '{desired_app_name}'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo """
                             PIPELINE STARTING...
                        Build Number: ${BUILD_NUMBER}
                        Image Tag: ${IMAGE_TAG}

                """
                sh 'git checkout main'
                sh 'ls -la'
            }
        }
        
        stage('Trivy Security Scan - Source Code') {
            steps {
                container('trivy') {
                    echo 'Scanning source code for vulnerabilities...'
                    sh '''
                        trivy fs --exit-code 0 --severity HIGH,CRITICAL --no-progress .
                        echo "Source code security scan completed!"
                    '''
                }
            }
        }
        
        stage('Build & Unit Test') {
            steps {
                container('maven') {
                    echo 'Building Spring Boot application...'
                    sh '''
                        mvn clean compile test -T 4C -q
                        echo "Build and tests completed!"
                    '''
                    junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Package Application - JAR') {
            steps {
                container('maven') {
                    sh 'mvn package -DskipTests'
                    sh 'ls -lh target/*.jar'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Docker Building Image') {
            steps {
                container('docker') {
                    echo 'Building Docker image...'
                    sh '''
                        # Wait for Docker daemon to be ready
                        echo "Waiting for Docker daemon..."
                        for i in $(seq 1 30); do
                            docker info >/dev/null 2>&1 && break
                            echo "Still waiting... ($i/30)"
                            sleep 2
                        done

                        echo "Building Docker image..."
                        docker build -t ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                        echo "Docker image built!"
                        docker images | grep ${IMAGE_NAME}
                    '''
                }
            }
        }
        
        stage('Trivy Security Scans Built Docker Image') {
            steps {
                container('trivy') {
                    echo 'Scanning Docker image for vulnerabilities...'
                    sh '''
                        # Wait for Docker daemon
                        echo "Connecting to Docker daemon..."
                        for i in $(seq 1 20); do
                            docker info >/dev/null 2>&1 && break
                            echo "Waiting for Docker connection... ($i/20)"
                            sleep 2
                        done

                        echo "Scanning image: ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        trivy image --exit-code 0 --severity HIGH,CRITICAL --no-progress ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                        echo "Docker image security scan completed!"
                    '''
                }
            }
        }
        
        stage('Push Built Image to Docker Hub') {
            steps {
                container('docker') {
                    echo 'Pushing to Docker Hub...'
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            echo "Pushing ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}..."
                            docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}
                            echo "Images pushed to Docker Hub!"
                        '''
                    }
                }
            }
        }
    } // end of stages
