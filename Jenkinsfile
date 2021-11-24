
// prerequisites: a nodejs app must be deployed inside a kubernetes cluster
// TODO: look for all instances of [] and replace all instances of 
//       '[VARIABLE]' with actual values 
//        e.g [GITREPO] might become https://github.com/MyName/internal.git
// variables:
//      [GITREPO]
//      [PROJECTID]
//      [APPNAME]
//      [CLUSTER_NAME] 
//      [ZONE]
//      the following values can be found in the yaml:
//      [DEPLOYMENT_NAME]
//      [CONTAINER_NAME] (name of the container to be replaced - in the template/spec section of the deployment)


pipeline {
    agent none 
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                }
            }
            steps {
                echo 'Retrieving source from github' 
                git branch: 'master',
                    url: 'https://github.com/LalitPatil08/sample-master-internal.git'
                echo 'Did we get the source?' 
                sh 'ls -a'
                echo 'install dependencies' 
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Tests passed on to build and deploy Docker container'
            }
        }
         stage('Build and deploy the container') {
             agent {
                docker { 
                    image 'google/cloud-sdk:latest'
                    args '-e HOME=/tmp'
                        }
                    }
            steps {
                echo "submit gcr.io/dtc-nov21-user122/internal-image:v2.${env.BUILD_ID}"
                sh "gcloud builds submit -t gcr.io/dtc-nov21-user122/internal-image:v2.${env.BUILD_ID} ."
                echo 'Get cluster credentials'
                sh 'gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project dtc-nov21-user122'
                echo "Update the image to use gcr.io/dtc-nov21-user122/internal-image:v2.${env.BUILD_ID}"
                sh "kubectl set image deployment/demo-api demo-api=gcr.io/dtc-nov21-user122/internal-image:v2.${env.BUILD_ID} --record"
            }
        }            
    }
}
