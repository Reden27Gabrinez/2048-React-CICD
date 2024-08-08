pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        // SCANNER_HOME=tool 'sonar-scanner'

        AWS_ACCOUNT_ID="558711665895"
        AWS_DEFAULT_REGION="ap-south-1"
        IMAGE_REPO_NAME="my-ecr-repo"
        // IMAGE_TAG="$BUILD_NUMBER"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "558711665895.dkr.ecr.ap-south-1.amazonaws.com/my-ecr-repo"

        cluster = "my-cluster"
        service = "my-service"
    }
    // environment {
    //     SCANNER_HOME=tool 'sonar-scanner'
    // }
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
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Reden27Gabrinez/2048-React-CICD.git'
            }
        }
        // stage("Sonarqube Analysis "){
        //     steps{
        //         withSonarQubeEnv('sonar-server') {
        //             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
        //             -Dsonar.projectKey=Game '''
        //         }
        //     }
        // }
        // stage("quality gate"){
        //   steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
        //         }
        //     }
        // }
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
        // stage("Docker Build and Push"){
        //   steps{
        //       script{
        //           withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //               sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
        //           }
        //           dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        //           sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"""
        //           sh """docker push ${REPOSITORY_URI}:${IMAGE_TAG}"""
        //       }
        //   }
        // }
        
        // Building Docker images
        stage('Building image') {
          steps{
            script {
              dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
          }
        }
        stage('Test Image'){
            steps{
                sh 'docker run -p 3000:3000 --name 2048 -d --rm ${IMAGE_REPO_NAME}:${IMAGE_TAG}'
                sh 'sleep  2'
                sh 'nc -zv localhost 3000'
                sh 'docker stop 2048'
            }
        } 
        
        stage('Pushing to ECR') 
        {
            steps
            {  
                script 
                {
                    sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                    sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image ${IMAGE_REPO_NAME}:${IMAGE_TAG} > trivy.txt"
            }
        }
        stage('Deploy to ECS') {
          steps {
            script{
                sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
            }
          }
        }
        
        


    }
}

// pipeline{
//     agent any
//     tools{
//         jdk 'jdk17'
//         nodejs 'node16'
//     }
//     environment {
//         SCANNER_HOME=tool 'sonar-scanner'

//         AWS_ACCOUNT_ID="558711665895"
//         AWS_DEFAULT_REGION="ap-south-1"
//         IMAGE_REPO_NAME="my-ecr-repo"
//         IMAGE_TAG="$BUILD_NUMBER"
//         REPOSITORY_URI = "558711665895.dkr.ecr.ap-south-1.amazonaws.com/my-ecr-repo"

//         cluster = "netflixapp-cluster"
//         service = "netflixapp-service"
//     }
//     stages {
//         stage('clean workspace'){
//             steps{
//                 cleanWs()
//             }
//         }
      
//         stage('Logging into AWS ECR') {
//           steps {
//               script {
//               sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
//               }   
//             }
//         }
          
//         stage('Checkout from Git'){
//             steps{
//                 git branch: 'main', url: 'https://github.com/Reden27Gabrinez/2048-React-CICD.git'
//             }
//         }
      
//         stage("Sonarqube Analysis "){
//             steps{
//                 withSonarQubeEnv('sonar-server') {
//                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
//                     -Dsonar.projectKey=Game '''
//                 }
//             }
//         }
      
//         stage("quality gate"){
//            steps {
//                 script {
//                     waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
//                 }
//             }
//         }
      
//         stage('Install Dependencies') {
//             steps {
//                 sh "npm install"
//             }
//         }
      
//         stage('OWASP FS SCAN') {
//             steps {
//                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
//                 dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
//             }
//         }
      
//         stage('TRIVY FS SCAN') {
//             steps {
//                 sh "trivy fs . > trivyfs.txt"
//             }
//         }

//         stage("Docker Build and Push"){
//           steps{
//               script{
//                   withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
//                       sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
//                   }
//                   dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
//                   sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:${IMAGE_TAG}"""
//                   sh """docker push ${REPOSITORY_URI}:${IMAGE_TAG}"""
//               }
//           }
//         }
//         stage("TRIVY"){
//             steps{
//                 sh "trivy image ${IMAGE_REPO_NAME}:${IMAGE_TAG} > trivy.txt"
//             }
//         }

//         stage('Deploy to container'){
//             steps{
//                 sh 'docker run -d --name 2048 -p 3000:3000 ${IMAGE_REPO_NAME}:${IMAGE_TAG}'
//             }
//         }

//         // stage('Deploy to ECS') {
//         //   steps {
//         //     script{
//         //       withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
//         //               sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
//         //       }
//         //       sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
//         //     }
//         //   }
//         // }
         
//     }
// }
