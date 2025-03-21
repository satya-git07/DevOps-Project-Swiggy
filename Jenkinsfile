pipeline {
    agent any

    environment {
        CLUSTER_NAME = 'my-cluster1'
        ZONE = 'us-west3'
        PROJECT_ID = 'lyrical-bus-452711-c5'
        GIT_BRANCH = 'master'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/satya-git07/DevOps-Project-Swiggy.git', branch: "${GIT_BRANCH}"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    sh """
                        /opt/sonar-scanner-new/sonar-scanner-4.8.0.2856-linux/bin/sonar-scanner \
                            -Dsonar.projectKey="swigy" \
                            -Dsonar.sources="." \
                            -Dsonar.host.url="http://192.168.2.179:9000" \
                            -Dsonar.login="squ_d23f6fdbd9cba7c7b641691e46d8f4593588cf94"
                    """
                }
            }
        }

      
stage("Docker Build & Push") {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-credentials', toolName: 'docker') {
                
                    sh "docker build -t swigy ."
                    sh "docker tag swigy:latest satyadockerhub07/swigy:tagname"
                    sh "docker push satyadockerhub07/swigy:tagname"
               
            }
        }
    }
}

          stage("TRIVY"){
                    steps{
                        sh "trivy image satyadockerhub07/swigy:tagname" 
                    }
                }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                       sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                withCredentials([file(credentialsId: 'gcp-sa', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
                    script {
                        dir('AgroCD') {
                        sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                        sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone ${ZONE} --project ${PROJECT_ID}"
                        sh "kubectl apply -f deploy.yaml"
                    }
                    }
                }
            }
        }
    }

    
}
