pipeline {
    agent any
    stages {
        stage('Linting') {
            steps {
                sh 'tidy -q -e *.html'
            }
        } 
        stage('Build the Docker image') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        docker build -t mohit062000/cloudcapstone .
                    '''
                }
            }
        }
        stage('Security Testing with Aqua') {
            steps { 
                aquaMicroscanner imageName: 'mohit062000/cloudcapstone', notCompliesCmd: 'exit 1', onDisallowed: 'fail', outputFormat: 'html'
            }
        }
        stage('Upload image to Docker') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                    sh '''
                        dockerpath="$DOCKER_USERNAME/cloudcapstone"
                        echo "Docker ID and Image: $dockerpath"
                        my_password="$DOCKER_PASSWORD"
                        echo "$my_password" | docker login --username $DOCKER_USERNAME --password-stdin
                        docker push $dockerpath
                    '''
                }
            }
        }

        stage('Create the kubernetes cluster') {
            steps {
                withAWS(region:'us-west-2',credentials:'awsdeploy') {
                    sh 'echo "Create k8s cluster..."'
                    sh '''
                        eksctl create cluster \
                        --name mohitcapstonecluster \
                        --version 1.16 \
                        --region us-west-2 \
                        --nodegroup-name standard-workers \
                        --node-type t2.micro \
                        --nodes 3 \
                        --nodes-min 1 \
                        --nodes-max 4 \
                        --managed
                    '''
                }
            }
        }

        stage('Create a configuration file for kubectl cluster') {
            steps {
                withAWS(region:'us-west-2',credentials:'awsdeploy') {
                    sh '''
                        aws eks --region us-west-2 update-kubeconfig --name mohitcapstonecluster
                    '''
                }
            }
        }
        
    }
}
