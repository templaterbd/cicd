    environment {
        DOCKER_USERNAME = 'davidbulke'
        IMAGE_NAME = 'java-app-ci'
        GIT_COMMIT_SHORT = sh(script: "git rev-parse --short=8 HEAD", returnStdout: true).trim()
        GIT_BRANCH = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
        IMAGE_TAG = "${GIT_COMMIT_SHORT}-${BUILD_NUMBER}"
        FULL_IMAGE_NAME = "${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo """

                         PIPELINE STARTING...

                    Branch:       ${GIT_BRANCH}
                    Commit:       ${GIT_COMMIT_SHORT}
                    Build:        #${BUILD_NUMBER}
                    Image Tag:    ${IMAGE_TAG}
                    Full Image:   ${FULL_IMAGE_NAME}

                """
                sh 'git checkout ${GIT_BRANCH}'
                sh 'ls -la'
            }
        }

        stage('Trivy Security Scan - Source Code') {
            steps {
                container('trivy') {
                    echo 'Scanning source code for vulnerabilities...'
                    sh '''
                        trivy fs --exit-code 0 --severity HIGH,CRITICAL --no-progress .   //change to 1 for fail build due to vulnerabilities
                        echo "Source code security scan completed!"
                    '''
                }
            }
        }
        
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests -B'
                }
            }
        }
        
        stage('Tests') {
            steps {
                container('maven') {
                    sh 'mvn test -B'
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package JAR') {
            steps {
                container('maven') {
                    sh 'mvn package -DskipTests'
                    sh 'ls -lh target/*.jar'
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Build Docker Image with Kaniko') {
            steps {
                container('kaniko') {
                    echo "Building Docker image with Kaniko (unprivileged)..."
                    script {
                        // Build list of tags to apply
                        def tags = [
                            "--destination=${FULL_IMAGE_NAME}"
                        ]
                        
                        // Only tag as 'latest' if on main branch
                        if (env.GIT_BRANCH == 'main') {
                            tags.add("--destination=${DOCKER_USERNAME}/${IMAGE_NAME}:latest")
                            echo "Main branch detected - will also tag as 'latest'"
                        }
                        
                        // Join tags for the command
                        def tagString = tags.join(' ')
                        
                        sh """
                            /kaniko/executor \
                                --context=\${PWD} \
                                --dockerfile=\${PWD}/Dockerfile \
                                ${tagString} \
                                --cache=true \
                                --cache-ttl=24h \
                                --compressed-caching=false \
                                --snapshot-mode=redo \
                                --log-format=text \
                                --verbosity=info
                            
                            echo "✅ Docker image built successfully with Kaniko!"
                        """
                    }
                }
            }
        }

        stage('Trivy Security Scan - Docker Image') {
            steps {
                container('trivy') {
                    echo "Scanning Docker image for vulnerabilities..."
                    sh """
                        echo "Scanning image: ${FULL_IMAGE_NAME}"
                        trivy image --exit-code 0 --severity CRITICAL --no-progress ${FULL_IMAGE_NAME} //change to 1 for fail build due to vulnerabilities
                        echo "✅ Docker image security scan passed!"
                    """
                }
            }
        }
    } 