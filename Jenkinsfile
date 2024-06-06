pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'

        AWS_ACCOUNT_ID="558711665895"
        AWS_DEFAULT_REGION="ap-south-1"
        IMAGE_REPO_NAME="my-ecr-repo"
        IMAGE_TAG="v1"
        REPOSITORY_URI = "558711665895.dkr.ecr.ap-south-1.amazonaws.com/my-ecr-repo"
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
      
       stage('Logging into AWS ECR') {
          steps {
              script {
              sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
              }   
        }
          
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Reden27Gabrinez/2048-React-CICD.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        // Building Docker images
        // stage('Building image') {
        //   steps{
        //     script {
        //       dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        //     }
        //   }
        // }
        stage("Docker Build"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image dockerImage > trivy.txt"
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
         steps{  
           script {
             withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
             }
           }
          }
        }
       }
         
    }
}
