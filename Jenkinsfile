pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        IMAGE_NAME = "sumantharya/bankapp"
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'git',
                    url: 'https://github.com/SumantharyaM/aws-devops-ci-pipeline.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Testing') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table ."
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=gcbank \
                        -Dsonar.projectName=gcbank \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false,
                        credentialsId: 'sonar-token'
                }
            }
        }

        stage('Build Package') {
            steps {
                sh "mvn package -DskipTests"
            }
        }

        stage('Publish To Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'devopsshack',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh "mvn deploy -DskipTests"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Check Local Images') {
            steps {
                sh "docker images | grep bankapp"
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh "trivy image ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([
                    credentialsId: 'docker-cred',
                    url: 'https://index.docker.io/v1/'
                ]) {
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Update Manifest File in Mega-Project-CD') {
            steps {
                cleanWs()

                withCredentials([
                    usernamePassword(
                        credentialsId: 'git',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh """
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/SumantharyaM/aws-devops-cd-k8s.git
                        cd aws-devops-cd-k8s

                        sed -i "s|sumantharya/bankapp:.*|${IMAGE_NAME}:${IMAGE_TAG}|" Manifest/manifest.yaml

                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"
                        git add Manifest/manifest.yaml
                        git commit -m "Update image tag to ${IMAGE_TAG}"
                        git push origin main
                    """
                }
            }
        }
    }

    post {
        always {
            script {
                def status = currentBuild.result ?: 'SUCCESS'
                def color = status == 'SUCCESS' ? 'green' : 'red'

                emailext(
                    subject: "${env.JOB_NAME} - Build ${env.BUILD_NUMBER} - ${status}",
                    body: """
                        <html>
                        <body>
                            <div style="border:4px solid ${color}; padding:10px;">
                                <h2>${env.JOB_NAME} - Build ${env.BUILD_NUMBER}</h2>
                                <div style="background-color:${color}; padding:10px;">
                                    <h3 style="color:white;">Pipeline Status: ${status}</h3>
                                </div>
                                <p>Check Console: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                            </div>
                        </body>
                        </html>
                    """,
                    to: '567adddi.jais@gmail.com',
                    from: 'jenkins@devopsshack.com',
                    mimeType: 'text/html'
                )
            }
        }
    }
}
