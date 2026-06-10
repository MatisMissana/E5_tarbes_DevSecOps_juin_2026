pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        APP_NAME    = "e5-tarbes-devsecops-app"
        TRIVY_EXE   = "C:\\ProgramData\\chocolatey\\bin\\trivy.exe"
        SNYK_EXE    = "C:\\ProgramData\\chocolatey\\bin\\snyk.exe"
    }

    parameters {
        string(name: 'NAME', defaultValue: 'DevOps User', description: 'Please tell me your name?')
        booleanParam(name: 'SKIP_TEST', defaultValue: false, description: 'Skip running Tests?')
        booleanParam(name: 'SKIP_SNYK', defaultValue: false, description: 'Skip Snyk scan?')
        choice(name: 'BRANCH', choices: ['Master', 'Dev'], description: 'Choose branch')
        password(name: 'SONAR_SERVER_PWD', description: 'Enter SONAR password')
    }

    stages {

        stage('01 - PRINT PARAMETERS') {
            steps {
                echo "Hello ${params.NAME}"
                echo "Skip tests: ${params.SKIP_TEST}"
                echo "Branch: ${params.BRANCH}"
            }
        }

        stage('02 - CHECK TOOLS') {
            steps {
                bat """
                echo ===== MAVEN =====
                mvn -version
                echo ===== TRIVY =====
                "%TRIVY_EXE%" --version
                echo ===== SNYK =====
                "%SNYK_EXE%" --version
                """
            }
        }

        stage('03 - BUILD') {
            steps {
                echo 'Compilation avec Maven...'
                bat 'mvn clean compile'
            }
        }

        stage('04 - TEST') {
            when {
                expression { return !params.SKIP_TEST }
            }
            steps {
                echo 'Tests unitaires...'
                bat 'mvn test'
            }
        }

        stage('05 - PACKAGE') {
            steps {
                echo 'Creation du JAR...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('06 - TRIVY SCAN') {
            steps {
                echo 'Scan de securite avec Trivy...'
                bat '"%TRIVY_EXE%" fs --severity HIGH,CRITICAL --exit-code 1 .'
            }
        }

        stage('07 - SNYK SCAN') {
            when {
                expression { return !params.SKIP_SNYK }
            }
            steps {
                echo 'Scan de securite avec Snyk...'
                withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
                    bat """
                    "%SNYK_EXE%" auth %SNYK_TOKEN%
                    "%SNYK_EXE%" test --severity-threshold=high
                    """
                }
            }
        }

        stage('08 - DEPLOY') {
            steps {
                echo 'Mise en Production !'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline termine avec succes !'
        }
        failure {
            echo '❌ Echec du pipeline - verifier les failles de securite !'
        }
    }
}