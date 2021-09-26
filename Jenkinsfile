pipeline {
    agent any
    stages {
        stage('Git-Checkout') {
            steps {
                echo 'Checking out from Git Repo';
                git branch: 'master', url: '{your-git-repo-web-url}'
            }
        }
        stage('Check Git Secrets') {
            steps {
                sh 'rm trufflehog || true'
                sh 'docker run -t gesellix/trufflehog --json {your-git-repo-web-url} > trufflehog'
                sh 'cat trufflehog'
            }
        }
        
        stage('SAST') {
            steps{
                withSonarQubeEnv('sonar') {
                        sh './gradlew sonarqube'
                }
            }
        }
      
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: 'scan="src" --format HTML', odcInstallation: 'OWASP'
            }
        }
       
        stage('Run JUnit tests') {
            steps {
                sh './gradlew test'
            }   
        }
        stage('Build JAR') {
            when {
                expression { choice == "build" }
                }
                steps {
                    sh './gradlew shadowJar'
                }
        }
        stage('Skip build') {
            when {
                expression { choice == "ignore" }
                }
                steps {
                    echo 'Did not build JAR';
                }
        } 
    }
}
