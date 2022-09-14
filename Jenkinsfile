pipeline{
    agent{
        label none
    }
    environment {
        DOCKER_IMAGE="tomtpc/nginx"
    }
    stages{
        stage("Build Image"){
            agent{
                label "docker"
            }
            options{
                timeout(time: 10, unit: 'MINUTES')
            }
            environment {
                DOCKER_TAG = "${GIT_BRANCH}-${BUILD_NUMBER}"
            }
            steps{
                echo "========Buiding A New Docker Image========"
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                echo "========Pushing Docker Image To Docker Hub========"
                withCredentials([usernamePassword(credentialsId: 'docker-hub', 
                                                  usernameVariable: 'DOCKER_USERNAME' , 
                                                  passwordVariable: 'DOCKER_PASSWORD')]) 
                {
                    sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
            post{
                success{
                    echo "========Build & Push Image Successfully========"
                }
                failure{
                    echo "========Build & Push Image failed========"
                }
            }
        }
        stage("Deploy to Servers"){
            agent{
                label "ansible"
            }
            options{
                timeout(time: 10, unit: 'MINUTES')
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub', 
                                                  usernameVariable: 'DOCKER_USERNAME' , 
                                                  passwordVariable: 'DOCKER_PASSWORD')]) 
                {
                    ansbilePlaybook(
                        credentialsId: 'private_key',
                        playbook: 'playbook.yml',
                        inventory: 'hosts',
                        become: 'yes', 
                        becomeUser: 'root',
                        extraVars: [
                            DOCKER_USERNAME: "${DOCKER_USERNAME}",
                            DOCKER_PASSWORD: "${DOCKER_PASSWORD}",
                            BRANCH: "${GIT_BRANCH.tokenize('/')[-1]}"
                        ]
                    )
                }
            }
        }
    }
    post{
        always{
            echo "========Pipeline's Result========"
        }
        success{
            echo "=> pipeline execution successfully"
        }
        failure{
            echo "=>pipeline execution failed"
        }
    }
}