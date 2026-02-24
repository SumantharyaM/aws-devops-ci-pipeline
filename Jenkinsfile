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
                sh "mvn compile"
            }
        }

        stage('Testing') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
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
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: false,
                        credentialsId: 'sonar-token'
                }
            }
        }

        stage('Build Package') {
            steps {
                sh "mvn package"
            }
        }

        stage('Publish To Nexus') {
            steps {
                withMaven(
                    globalMavenSettingsConfig: 'devopsshack',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh "mvn deploy"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh "trivy image --format table -o image-report.html ${IMAGE_NAME}:${IMAGE_TAG}"
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
                        git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/jaiswaladi246/Mega-Project-CD.git
                        cd Mega-Project-CD

                        sed -i "s|sumantharya/bankapp:.*|${IMAGE_NAME}:${IMAGE_TAG}|" Manifest/manifest.yaml

                        echo "Updated manifest file:"
                        cat Manifest/manifest.yaml

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
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'SUCCESS'
                def bannerColor = pipelineStatus == 'SUCCESS' ? 'green' : 'red'

                def body = """
                <html>
                <body>
                <div style="border:4px solid ${bannerColor}; padding:10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color:${bannerColor}; padding:10px;">
                        <h3 style="color:white;">Pipeline Status: ${pipelineStatus}</h3>
                    </div>
                    <p>Check the <a href="${env.BUILD_URL}">Console Output</a></p>
                </div>
                </body>
                </html>
                """

                emailext(
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus}",
                    body: body,
                    to: '567adddi.jais@gmail.com',
                    from: 'jenkins@devopsshack.com',
                    mimeType: 'text/html'
                )
            }
        }
    }
}
