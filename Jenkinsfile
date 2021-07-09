pipeline {
    agent any
    stages {
        stage('Git-Checkout') {
            steps {
                echo 'Checking out from Git Repo';
                git branch: 'master', url: 'https://github.com/mabel-kang/ip.git'
            }
        }
        stage('Check Git Secrets') {
            steps {
                sh 'rm trufflehog || true'
                sh 'docker run -t gesellix/trufflehog --json https://github.com/mabel-kang/ip.git > trufflehog'
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
            steps {
                sh './gradlew shadowJar'
            }
        }
    }
}
