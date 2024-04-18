pipeline {
    agent any
    environment {
        registry = "288214676350.dkr.ecr.ap-south-1.amazonaws.com/myecrrepo"
        dockerImageTag = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Code Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/peddiredds/jenkinscicd.git']])
            }
        }
        stage('Build and Test') {
            steps {
               sh 'ls -ltr'
               sh 'mvn clean package'
            }
        }
        stage('Static Code Analysis') {
        environment {
            SONAR_URL = "http://3.109.186.70:9000"
              }
         steps {
             withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_AUTH_TOKEN')]) {
               sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
            }
           }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${dockerImageTag}")
                }
            }
        }
        stage('Pushing to ECR') {
            steps {
                script {

                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 288214676350.dkr.ecr.ap-south-1.amazonaws.com'
                    sh "docker push ${registry}:${dockerImageTag}"
                }
            }
        }
        stage('Update Helm Chart') {
            environment {
               GIT_REPO_NAME = "peddiredds/argocd.git"
               GIT_USER_NAME = "peddiredds"
               helmChartPath = "argocd/charts/myapp"
             }
            steps {
                withCredentials([string(credentialsId: 'git-credentials-id', variable: 'GITHUB_TOKEN')]) {
                    sh'''
                      git config user.email "srinivasreddy.skim@gmail.com"
                      git config user.name "srinivas"
                      rm -rf argocd
                      git clone https://${GIT_USER_NAME}:${GITHUB_TOKEN}@github.com/${GIT_REPO_NAME}
                      cd argocd
                      git pull
                      cd ..
                      sed -i "s/dockerImageTag: .*/dockerImageTag: \"${dockerImageTag}\"/" ${helmChartPath}/values.yaml
                      cd argocd
                      pwd
                      git add charts/myapp/values.yaml
                      git commit -m "Update image tag in values.yaml"
                      git push https://${GITHUB_TOKEN}@github.com/${GIT_REPO_NAME} HEAD:main
                 '''
        }
    }
}

    }
}