pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    parameters {
        string(name: 'NAME', description: 'Please tell me your name?')
        booleanParam(name: 'SKIP_TEST', description: 'Want to skip running Test cases?')
        choice(name: 'BRANCH', choices: ['Master', 'Dev'], description: 'Choose branch')
        password(name: 'SONAR_SERVER_PWD', description: 'Enter SONAR password')
    }

    stages {

        stage('Printing Parameters') {
            steps {
                echo "Hello ${params.NAME}"
                echo "Skip tests: ${params.SKIP_TEST}"
                echo "Branch: ${params.BRANCH}"
            }
        }

        stage('BUILD') {
            steps {
                echo 'Compilation avec Maven...'
                bat 'mvn clean compile'
            }
        }

        stage('Test') {
            when {
                expression { return !params.SKIP_TEST }
            }
            steps {
                echo 'Tests unitaires...'
                bat 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Creation du JAR...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('Security Scan') {
            steps {
                echo 'Scan de securite avec Trivy...'
                bat 'trivy fs --severity HIGH,CRITICAL --exit-code 1 .'
            }
        }

        stage('DEPLOY') {
            steps {
                echo 'Mise en Production !'
            }
        }
    }

    post {
        success {
            echo 'Pipeline termine avec succes !'
        }
        failure {
            echo 'Echec du pipeline - verifier les failles de securite !'
        }
    }
}