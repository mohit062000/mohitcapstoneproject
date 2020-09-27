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

        stage('Create the blue replication controller with its docker image') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
                    sh '''
                        kubectl apply -f ./blue-controller.json
                    '''
                }
            }
        }
        stage('Create the green replication controller with its docker image') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
                    sh '''
                        kubectl apply -f ./green-controller.json
                    '''
                }
            }
        }
        stage('Create the service in kubernetes cluster to the blue replication controller') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
                    sh '''
                        # Create the service in kubernetes cluster in charge of routing traffic
                        # to the blue replication controller and exposing it to the outside
                        # world by setting the selector to app=blue
                        kubectl apply -f ./blue-service.json
                    '''
                }
            }
        }
        stage('Wait until the user gives the instruction to continue') {
            steps {
                input('Does the blue deployment look ok, and do you want to deploy to green?')
            }
        }
        stage('Update the service to redirect to green by changing the selector to app=green') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
                    sh '''
                        kubectl apply -f ./green-service.json
                    '''
                }
            }
        }
        stage('Check the application deployed in the cluster and its correct deployment') {
            steps {
                withAWS(region:'us-west-2',credentials:'aws-static') {
                    sh '''
                        # Verify deployed application is running. Existing pods must be running.                        # must be running
                        kubectl get pods
                        # Obtain information and IP of the service. It can be found as 
                        # "LOAD_BALANCER_INGRESS + : + PORT", and subsequently accessed from
                        # the browser. 
                        kubectl describe service bluegreenlb
                    '''
                }
            }
        }
        
    }
}
