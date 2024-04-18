pipeline {
    agent any
    tools{
        jdk 'jdk21'
        nodejs 'nodejs'
    }
     environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('code checkout') {
            steps {
                git branch: 'main', url: "https://github.com/ajjurathod/Netflix-clone.git"
            } 
            
        }
        stage("sonarqube  analysis"){
            steps{
                sh "npm install"
                withSonarQubeEnv('sonar-server'){
               sh ''' $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=ntflix2 \
                    -Dsonar.projectKey=ntflix2 '''
                       
                }   
              }
        }
        stage("quality gatecheck"){
            steps{
               waitForQualityGate abortPipeline: false, credentialsId: 'sonar-scanner' 
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("containerzation"){
            steps{
                sh  "docker build --build-arg TMDB_V3_API_KEY=ce28e87c0a141e10489ca0af14651c89 -t ajjurathod1998/netflix:latest ."
            }
        }
        stage("pushing image to dockerhub"){
            steps{
                withCredentials([string(credentialsId: 'docker-tocken', variable: 'DockerHubPwd')]) {
               sh "docker login -u ajjurathod1998 -p ${DockerHubPwd}"
               sh "docker push ajjurathod1998/netflix:latest"
               }
            }
        }
         stage("TRIVY image scan"){
            steps{
                sh "trivy image ajjurathod1998/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }   
                    }
                }
            }
        }
        post {
          always {
              emailext attachLog: true,
              subject: "'${currentBuild.result}'",
              body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
              to: 'ajjurathod1998@gmail.com', 
              attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
        
    }
}
