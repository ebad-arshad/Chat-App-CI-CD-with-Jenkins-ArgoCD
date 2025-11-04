@Library('shared') _

pipeline {
    agent { label 'slave' }

    stages {
        stage('Version Calculation') {
            steps {
                script {
                    // Get current build number
                    def buildNumber = env.BUILD_NUMBER.toInteger()

                    // Calculate major and minor versions
                    def majorVersion = (buildNumber / 50).intValue()
                    def minorVersion = buildNumber % 50

                    // Format minor version with leading zero if needed
                    def formattedMinor = String.format('%02d', minorVersion)

                    // Create the tag
                    def imageTag = "${majorVersion}.${formattedMinor}"

                    // Store in environment variable
                    env.IMAGE_TAG = imageTag.toString()
                }
            }
        }
        stage('Clone') {
            steps {
                script {
                    sh "rm -rf main k8s-helm"

                    clone('https://github.com/ebad-arshad/Chat-App-CI-CD-with-Jenkins-ArgoCD.git', 'main')
                    // git clone -b <branch> <repo> <branch>

                    clone('https://github.com/ebad-arshad/Chat-App-CI-CD-with-Jenkins-ArgoCD.git', 'k8s-helm')
                    // git clone -b <branch> <repo> <branch>
                }
            }
        }
        stage('Login DockerHub') {
            steps {
                script {
                    dockerhub_login('docker-hub-creds')
                }
            }
        }
        stage('Build') {
            steps {
                echo 'This is build stage'
                sh "docker build -t ebadarshad/chat-frontend:${env.IMAGE_TAG} main/frontend"
                sh "docker build -t ebadarshad/chat-backend:${env.IMAGE_TAG} main/backend"
                echo 'Build images successful'
            }
        }
        stage('Security Scan with Trivy') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL --exit-code 1 --format json -o trivy-report.json ebadarshad/chat-frontend:${env.IMAGE_TAG}"
                sh "trivy image --severity HIGH,CRITICAL --exit-code 1 --format json -o trivy-report.json ebadarshad/chat-backend:${env.IMAGE_TAG}"
                archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            }
        }
        stage('Push image to DockerHub') {
            steps {
                echo 'This is Dockerhub image push stage'
                sh "docker push ebadarshad/chat-frontend:${env.IMAGE_TAG}"
                sh "docker push ebadarshad/chat-backend:${env.IMAGE_TAG}"
                echo 'Pushed image to Dockerhub'
            }
        }
        stage('Update Helm Manifests of k8s') {
            steps {
                script {
                    sh """
                    yq eval '.chat-frontend.image.tag = \"${env.IMAGE_TAG}\"' | .chat-backend.image.tag = \"${env.IMAGE_TAG}\"' k8s-helm/values.yaml
                    """
                    sh 'cat k8s-helm/values.yaml'
                }
            }
        }
        stage('Push Helm Manifests of k8s on GitHub') {
            steps {
                script {
                    // Commit and push to GitHub
                    withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh """
                            git config user.email "m.ebadarshad2003@gmail.com"
                            git config user.name "ebad-arshad"
                            git add values.yaml
                            if git diff --cached --quiet; then
                                echo "No changes to commit"
                            else
                                git commit -m "ci: update image tags to ${env.IMAGE_TAG}"
                                git push https://${GIT_TOKEN}@github.com/ebad-arshad/Chat-App-CI-CD-with-Jenkins-ArgoCD.git
                            fi
                        """
                    }
                }
            }
        }
    }
}
