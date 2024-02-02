

pipeline {
    environment {
        DOCKER_ID = "m0ph"
        DOCKER_IMAGE_USER_DB = "user-db"
        DOCKER_TAG = "${BUILD_ID}"
        BUILD_AGENT  = ""
        NAMESPACE = "sock-shop"
    }
agent any
    stages {
        stage('Build') {
            steps { //create a loop somehow??
            sh 'docker build -t $DOCKER_ID/$DOCKER_IMAGE_USER_DB:$DOCKER_TAG .'
                   }
        }
        stage('Run') {
            steps {
                sh 'docker network create $BUILD_TAG'
                sh 'docker run -d --name $DOCKER_IMAGE_USER_DB --rm --network $BUILD_TAG $DOCKER_ID/$DOCKER_IMAGE_USER_DB:$DOCKER_TAG'
                sh 'docker stop  $DOCKER_IMAGE_USER_DB'
                sh 'docker network rm $BUILD_TAG'    
                }
        }
        stage('Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
         steps {
            sh 'docker image tag $DOCKER_ID/$DOCKER_IMAGE_USER_DB:$DOCKER_TAG $DOCKER_ID/$DOCKER_IMAGE_USER_DB:latest'
            sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
            sh 'docker push $DOCKER_ID/$DOCKER_IMAGE_USER_DB:$DOCKER_TAG && docker push $DOCKER_ID/$DOCKER_IMAGE_USER_DB:latest'
            }
        }
        stage('Deploy EKS') {
            environment {
                KUBECONFIG = credentials("EKS_CONFIG")  
                AWSKEY = credentials("AWS_KEY")
                AWSSECRETKEY = credentials("AWS_SECRET_KEY")
                EKSCLUSTERNAME = credentials("EKS-CLUSTER")
                

            }
            steps{
                sh 'rm -Rf .kube'
                sh 'mkdir .kube'
                sh 'touch .kube/config'
                sh 'sudo chmod 777 .kube/config'
                //sh 'cat $KUBECONFIG > .kube/config'
                sh 'rm -Rf .aws'
                sh 'mkdir .aws'
                sh 'aws configure set aws_access_key_id $AWSKEY'
                sh 'aws configure set aws_secret_access_key $AWSSECRETKEY'
                sh 'aws configure set region eu-west-3'
                sh 'aws configure set output text'                
                sh 'aws eks --region eu-west-3 update-kubeconfig --name $EKSCLUSTERNAME --kubeconfig .kube/config'
                sh 'aws eks list-clusters'
                sh 'kubectl config view'
                sh 'kubectl cluster-info --kubeconfig .kube/config'
                sh 'kubectl apply -f ./manifests -n $NAMESPACE'
                }
            

        }
    }
}