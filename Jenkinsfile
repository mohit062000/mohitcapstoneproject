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
        
        stage('Scanning Image') {
        steps {
            sysdig engineCredentialsId: 'sysdig-secure-api-credentials', name: 'mohit062000/cloudcapstone', inlineScanning: true
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
