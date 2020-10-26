pipeline {
    agent {
        label 'gitops-jenkins-jenkins-slave'
    }
    stages {
        stage('Get Source') {
            steps {
                echo "1.Clone Repo Stage"
                git credentialsId: 'GitHubAccess', url: 'https://github.com/tianyuan848192/app-repo'
                script {
                    build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                    repo_name = '184287144747.dkr.ecr.us-east-2.amazonaws.com'
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
                withDockerRegistry(credentialsId: 'ecr:us-east-2:AWS-AKSK', url: 'https://184287144747.dkr.ecr.us-east-2.amazonaws.com/gitops-app-demo') {
                    sh "docker push ${repo_name}/${app_name}:latest"
                    sh "docker push ${repo_name}/${app_name}:${build_tag}"
                }
            }
        }
        stage('Clone and change apply file') {
            steps {
                echo "4.Clone Repo Stage"
                git credentialsId: 'GitHubAccess', url: 'https://github.com/tianyuan848192/config-repo'
                    sh "sed -i 's@TAG@'"${build_tag}"'@' workloads/gitops-example-dep.yaml"                    
                    sh "git clean -df"
                    sh "git add ./"
                    sh "git commit -m 'change tags'"
                    sh "git push origin HEAD:refs/heads/master"
            }
        }
    }
}
