pipeline {
    agent any 
    
    tools{
        maven 'maven'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'master', url: 'https://github.com/eyaboubaker/projet.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
       
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'DP-Check'
            }
        }
        
        stage('Publish OWASP Dependency Check Report') {
            steps {
                publishHTML(target: [
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'target',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
            }
        }
        
          stage('Build') {
            steps {
               sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker build -t boubakereya22/pet-clinic123:latest . "
                    }
               }
            }
        }
        
       stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html  boubakereya22/pet-clinic123:latest "
            }
        }
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                            sh "docker push boubakereya22/pet-clinic123:latest"
                    }
               }
            }
        }
        stage('Deploying Docker Image'){
        steps{
            script{
            withDockerRegistry(credentialsId: 'docker'){
                sh 'docker run -d --name petclinic -p 8082:8082 boubakereya22/pet-clinic123:latest'}
            }
        }
        }
    }
}
