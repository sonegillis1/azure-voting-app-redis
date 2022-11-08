pipeline {
    agent any
    stages {
        stage ('Verify Branch') {
            steps {
                echo "$GIT_BRANCH"
            }
        }
        stage ('Docker Build') {
            steps {
                sh(script: 'docker images -a')
                sh(script: """
                    cd azure-vote
                    docker images -a
                    docker build -t jenkins-pipeline .
                    docker images -a
                    cd ..
                """)
            }
        }
        stage('Start test app') {
            steps {
                sh(script: """
                    docker-compose down
                    docker-compose up -d
                    chmod +x ./scripts/test_container.sh
                    ./scripts/test_container.sh
                """)
            }
            post {
                success {
                    echo "App started successfully :)"
                }
                failure {
                    sh(script: 'docker-compose down')
                    echo "App failed to start :("
                }
            }
        }
        stage('Run Tests') {
            agent {
                docker 'qnib/pytest'
            }
            steps {
                sh(script: """
                    pytest ./tests/test_sample.py
                """)
            }
        }
        stage('Stop test app') {
            steps {
                sh(script: """
                    docker-compose down
                """)
            }
        }
        stage('Push Container') {
            steps {
                echo "Workspace is $WORKSPACE"
                dir("$WORKSPACE/azure-vote") {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
                            def image = docker.build('sonegillis/jenkins-course:latest')
                            image.push()
                        }
                    }
                }
            }
        }
        stage('Scan Container') {
            steps {
                sh(script: """
                    sudo apt-get install wget apt-transport-https gnupg lsb-release
                    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
                    echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
                    sudo apt-get update
                    sudo apt-get install trivy
                    trivy image sonegillis/jenkins-course:latest
                """)
            }
        }
    }
}
