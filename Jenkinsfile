pipeline {
    agent any

    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/andreybaton/cicd-demo-app', description: 'HTTPS-ссылка на Git-репозиторий')
        string(name: 'GIT_BRANCH', defaultValue: 'master', description: 'Ветка для сборки')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Выполнять unit-тесты')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.GIT_BRANCH}", url: "${params.REPO_URL}"
                script {
                    env.COMMIT_SHA = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                }
                echo "Ветка: ${params.GIT_BRANCH}"
                echo "Полный SHA коммита: ${env.COMMIT_SHA}"
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -f README.md || { echo "Ошибка: отсутствует README.md"; exit 1; }
                    test -f requirements.txt || { echo "Ошибка: отсутствует requirements.txt"; exit 1; }
                    test -d src || { echo "Ошибка: отсутствует каталог src/"; exit 1; }
                    test -d tests || { echo "Ошибка: отсутствует каталог tests/"; exit 1; }
                    echo "Валидация структуры проекта пройдена успешно."
                '''
            }
        }

        stage('Test') {
            when {
                expression { return params.RUN_TESTS == true }
            }
            steps {
                sh 'python3 -m unittest discover -s tests -v'
            }
        }

        stage('Build') {
            steps {
                sh """
                    rm -f app.tar.gz build-info.txt
                    echo "Build Number: ${env.BUILD_NUMBER}" > build-info.txt
                    echo "Commit SHA: ${env.COMMIT_SHA}" >> build-info.txt
                    tar -czf app.tar.gz src tests README.md requirements.txt build-info.txt
                """
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'app.tar.gz', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Сборка #${env.BUILD_NUMBER} успешно завершена!"
        }
        failure {
            echo "Сборка #${env.BUILD_NUMBER} завершилась с ошибкой (FAILURE)."
        }
    }
}
