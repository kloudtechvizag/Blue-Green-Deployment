pipeline {
    agent any
    
  
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['blue', 'green'], description: 'Choose which environment to deploy: Blue or Green')
        choice(name: 'DOCKER_TAG', choices: ['blue', 'green'], description: 'Choose the Docker image tag for the deployment')
        booleanParam(name: 'SWITCH_TRAFFIC', defaultValue: false, description: 'Switch traffic between Blue and Green')
    }
    
    environment {
        IMAGE_NAME = "vsiraparapu/bankapp"
        TAG = "${params.DOCKER_TAG}"
        KUBE_NAMESPACE = 'webapps'
        // SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

     
       
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Tests') {
            steps {
                sh "mvn clean test -X -DskipTests=true"
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
 
        stage('Docker Build & tag image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'venkat-docker-creds') {
                        sh "docker build -t ${IMAGE_NAME}:${TAG} ."
                    }
                }
            }
        }
        
        
        stage('Docker Push image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'venkat-docker-creds') {
                        sh "docker push ${IMAGE_NAME}:${TAG}"
                    }
                }
            }
        }

        stage('Deploy MySQL to Local K8s') {
            steps {
                withKubeConfig(credentialsId: 'venkat-kubect-config-creds') {
                    sh 'kubectl apply -f mysql-ds.yml'
                }
            }
        }


        stage('Deploy SVC app') {
            steps {
                withKubeConfig(credentialsId: 'venkat-kubect-config-creds') {
                    sh ' kubectl apply -f bankapp-service.yml'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentFile = ""
                    if (params.DEPLOY_ENV == 'blue') {
                        deploymentFile = 'app-deployment-blue.yml'
                    } else {
                        deploymentFile = 'app-deployment-green.yml'
                    }
                    
                    withKubeConfig(credentialsId: 'venkat-kubect-config-creds') {
                       sh "kubectl apply -f ${deploymentFile}"
                    }
                }
            }
        }


        
        
      
     
    }
}
