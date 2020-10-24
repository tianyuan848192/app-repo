pipeline {
    agent {
        label 'gitops-jenkins-jenkins-slave'
    }
    stages {
        stage('Get Source') {
            steps {
                echo "1.Clone Repo Stage"
                git credentialsId: 'github', url: 'https://github.com/tianyuan848192/app-repo'
                script {
                    build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    repo_name = '184287144747.dkr.ecr.us-east-2.amazonaws.com.cn'
                    app_name = 'gitops-app-demo'
                }
            }
        }
        stage('Build Image') {
            steps {
                echo "2.Build Docker Image Stage"
                sh "docker build --network host -t ${repo_name}/${app_name}:latest ."
                sh "docker tag ${repo_name}/${app_name}:latest ${repo_name}/${app_name}:${build_tag}"
            }

        }
        stage('Push Image') {
            steps {
                echo "3.Push Docker Image Stage"
                withDockerRegistry(credentialsId: 'ecr:cn-north-1:ecr-repo', url: 'https://184287144747.dkr.ecr.cn-north-1.amazonaws.com.cn/gitops-app') {
                    sh "docker push ${repo_name}/${app_name}:latest"
                    sh "docker push ${repo_name}/${app_name}:${build_tag}"
                }
            }
        }
    }
}
