pipeline {
    agent {
        label 'linux'
    }

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                echo 'Running Maven tests from the repository root'
                sh 'mvn clean test'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging the application from the repository root'
                sh 'mvn package'
            }
        }
    }

    post {
        always {
            echo 'Publishing JUnit test results'
            junit 'course-registration-app/target/surefire-reports/TEST-*.xml'
        }

        success {
            echo 'Archiving the packaged application'
            archiveArtifacts(
                artifacts: 'course-registration-app/target/*.jar',
                fingerprint: true,
                onlyIfSuccessful: true
            )
        }

        failure {
            echo 'Pipeline failed. Check the console output and test reports.'
        }
    }
}