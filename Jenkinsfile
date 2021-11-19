pipeline{
    agent any
    tools{
        maven 'MAVEN'
    }
    environment {
        GIT_TAG = "build-$BUILD_NUMBER"
    }
    stages{
        stage('build war'){
            steps{
                echo "Building war file"
                sh 'mvn clean package'
            }
        }
        stage('create git tag'){
            steps{
                sshagent (credentials: ['mac-ssh-key']) {
                    // sh 'git add .'
                    // sh('git commit -m "bumped to $GIT_TAG"')
                    sh "git tag -a $GIT_TAG -m 'Version $BUILD_NUMBER'"
                    sh('git push git@github.com:abhishek0795/springboot_maven.git HEAD:$BRANCH_NAME --tag')
                
                }
            }
        }
        stage('build and push docker image'){
            steps{
                withCredentials([
                    usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')
                ]){
                    sh('/Applications/Docker.app/Contents/Resources/bin/docker build -t $USERNAME/java-mvn:$GIT_TAG .')
                    // don't use this
//                     sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"   
                    sh('echo $PASSWORD | /Applications/Docker.app/Contents/Resources/bin/docker login -u $USERNAME --password-stdin')
                    sh('/Applications/Docker.app/Contents/Resources/bin/docker push $USERNAME/java-mvn:$GIT_TAG')
                }
            }
        }
        stage('deploy'){
            steps{
                echo "Deploying application"
                sh '/Applications/Docker.app/Contents/Resources/bin/docker rm -f java-mvn-app'
                sh ('/Applications/Docker.app/Contents/Resources/bin/docker run --rm -dp 4444:8080 --name java-mvn-app abhishekjha95/java-mvn:$GIT_TAG')
                echo "Application is live on <ip-address>:4444"
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '**/target/*.war', followSymlinks: false
        }
    }
}

