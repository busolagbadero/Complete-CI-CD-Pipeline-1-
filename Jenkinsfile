pipeline {
    agent any
    tools {
        maven 'maven-b'
    }
    stages {
        stage("increment Version") {
            steps {
                script {
                    echo 'incrementing version'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage("Build App") {
            steps {
                script {
                    echo 'building the app'
                    sh 'mvn clean package'
                }
            }
        }
        stage('Build Image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t gbaderobusola/busola:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push gbaderobusola/busola:${IMAGE_NAME}"
                    }
                }
            }
        }

        stage('Deploy_Env') {
            environment{
                AWS_ACCESS_KEY_ID = credentials('Jenkins-AWS-ACCESS-KEY')
                AWS_SECRET_ACCESS_KEY = credentials('Jenkins-AWS-SECRET-ACCESS-KEY')
                APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo "Deploying Docker Image"
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                   
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git config user.email "jenkins@example.com"'
                        sh 'git config user.name "Jenkins"'
                        sh "git remote set-url origin https://"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}

        