pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: gradle
    image: gradle:jdk21 // Используем образ с JDK 21, так как проект требует Java 25 (пока используем совместимый)
    command:
    - cat
    tty: true
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
'''
        }
    }

    stages {
        stage('Checkout') {
            steps {
                // Клонируем код из твоего публичного форка
                git url: 'https://github.com/TemaKham/spring-boot-with-postgres-example.git', branch: 'main'
            }
        }

        stage('Build (Skip Integration Tests)') {
            steps {
                container('gradle') {
                    // Собираем проект, пропуская интеграционные тесты, чтобы избежать проблем с Testcontainers в K8s
                    sh './gradlew clean build -x test --no-daemon'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                container('gradle') {
                    // Запускаем только unit-тесты (если они есть и не требуют БД)
                    // Если таких нет, этот этап можно будет убрать или изменить
                    sh './gradlew test --no-daemon -Dtest=*Test'
                }
            }
        }
    }

    post {
        always {
            // Публикуем результаты тестов (если они были)
            junit allowEmptyResults: true, testResults: 'build/test-results/test/*.xml'
            // Сохраняем собранный артефакт
            archiveArtifacts artifacts: 'build/libs/*.jar', allowEmptyArchive: true, fingerprint: true
        }
    }
}
