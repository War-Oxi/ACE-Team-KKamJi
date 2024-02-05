pipeline {
    agent any

    tools {
        go 'kkam_go'
    }

    environment {
        GITNAME = 'war-oxi'
        GITMAIL = 'xowl5460@naver.com'
        GITWEBADD = 'https://github.com/War-Oxi/ACE-Team-KKamJi.git'
        GITSSHADD = 'git@github.com:War-Oxi/ACE-Team-KKamJi.git'
        GITCREDENTIAL = 'kkam_git_cre'

        ECR_URL = '109412806537.dkr.ecr.us-east-1.amazonaws.com/app-picture-backend'
        ECR_CREDENTIAL = 'aws_cre'
    }

    stages {
        stage('Checkout Github') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [],
                userRemoteConfigs: [[credentialsId: GITCREDENTIAL, url: GITWEBADD]]])
            }
            post {
                failure {
                    echo 'Repository clone failure'
                }
                success {
                    echo 'Repository clone success'
                }
            }
        }

        stage('image build') {
            steps {
                sh "docker build -t ${ECR_URL}:${currentBuild.number} ."
                sh "docker build -t ${ECR_URL}:latest ."
            }
        }

        stage('image push') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY', credentialsId: ECR_CREDENTIAL]]) {
                    script {
                        def ecrLogin = sh(script: "aws ecr get-login-password --region us-east-1", returnStdout: true).trim()
                        sh "docker login -u AWS -p ${ecrLogin} ${ECR_URL}"
                    }
                }
                sh "docker push ${ECR_URL}:${currentBuild.number}"
                sh "docker push ${ECR_URL}:latest"
            }
            post {
                failure {
                    echo 'AWS ECR로 이미지 푸시 실패'
                    sh "docker image rm -f ${ECR_URL}:${currentBuild.number}"
                    sh "docker image rm -f ${ECR_URL}:latest"
                }
                success {
                    echo 'AWS ECR로 이미지 푸시 성공'
                    sh "docker image rm -f ${ECR_URL}:${currentBuild.number}"
                    sh "docker image rm -f ${ECR_URL}:latest"
                }
            }
        }

        stage('k8s manifest file update') {
            steps {
                git credentialsId: GITCREDENTIAL,
                url: GITSSHADD,
                branch: 'main'

                sh "git config --global user.email ${GITMAIL}"
                sh "git config --global user.name ${GITNAME}"
                sh "sed -i 's@${ECR_URL}:.*@${ECR_URL}:${currentBuild.number}@g' ingress/app_group/picture_backend/picture_backend_deployment.yml"
                sh "git add ."
                sh "git commit -m 'fix:${ECR_URL} ${currentBuild.number} image versioning'"
                sh "git branch -M main"
                sh "git remote remove origin"
                sh "git remote add origin ${GITSSHADD}"
                sh "git push -u origin main"
            }
            post {
                failure {
                    echo 'k8s manifest file update failure'
                }
                success {
                    echo 'k8s manifest file update success'
                }
            }
        }
    }
}
