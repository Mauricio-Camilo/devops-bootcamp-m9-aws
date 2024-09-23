#!/user/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://github.com/Mauricio-Camilo/jenkins-shared-library.git',
    credentialsId: 'github-credentials'])

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    // environment {
    //     IMAGE_NAME = 'mauriciocamilo/demo-app:java-maven-2.0'
    // }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'increment app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                    -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                    versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "mauriciocamilo/demo-app:java-maven-$version-$BUILD_NUMBER"
                }
            }
        } 
        stage("build app") {
            steps {
                script {
                    echo 'building application jar...'
                    buildJar()
                }
            }
        }

        stage("build image") {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}" 
                    def ec2Instance = "ec2-user@54.211.179.187"
                    sshagent(['ec2-server-key']) {
                        sh "scp server-cmds.sh ${ec2Instance}:/home/ec2-user"
                        sh "scp docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                    }
                }
            }
        } 

         stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        echo "User: ${USER}"
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'

                        // Replace '@' in email with '%40' for URL encoding
                        def encodedUser = USER.replace('@', '%40')
                
                        // Set the remote URL using encoded user
                        sh "git remote set-url origin https://${encodedUser}:${PASS}@github.com/Mauricio-Camilo/java-maven-app"

                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-version'
                    }
                }
            }
        }

    }
} 