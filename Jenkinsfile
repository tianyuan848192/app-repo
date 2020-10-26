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
        stage("pull config repo") {
            steps {
                echo "clone repo"
                git url: "https://github.com/tianyuan848192/config-repo",
                    credentialsId: "GitHubAccess",
                    branch: "master"                
            }
        }
        stage("add tag to file") {
            steps {
                echo "change file"
                sh 'sed -i \'s@TAG@\'"${build_tag}"\'@\' workloads/gitops-example-dep.yaml'
            }
        }
        stage("push to repo") {
            steps {
                withCredentials([usernamePassword(credentialsId:"GitHubAccess",
                                                  usernameVariable: "GIT_USERNAME", 
                                                  passwordVariable: "GIT_PASSWORD")]) {
                    sh 'git config --local credential.helper "!p() { echo username=\\$GIT_USERNAME; echo password=\\$GIT_PASSWORD; }; p"'
                    sh 'git config --global user.name "tianyuan"'
                    sh 'git config --global user.email "tianyuan848192@hotmail.com"'
                    sh 'git add ./'
                    sh 'git commit -m "add tag"'
                    sh 'git push -u origin master'
                }
            }
        }
    }
}
