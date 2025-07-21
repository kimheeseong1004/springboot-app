pipeline {
    agent any

    tools {
        // Jenkins 설정과 일치하는 JDK, Gradle 이름 사용
        jdk 'JDK17'
        gradle 'gradle' 
    }

    environment {
        // Jenkins 파이프라인 설정에서 이 변수들의 값을 지정해야 합니다.
        // 예: DB_USER = credentials('db-credentials-id')
        DB_HOST = "localhost:3306"
        DB_NAME = "myapp"
        DB_USER = "jenkins"
        DB_PASS = "jenkins123"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/kimheeseong1004/springboot-app.git'
            }
        }

        stage('Build') {
            steps {
                bat 'gradle clean build'
            }
        }

        stage('Test') {
            steps {
                bat 'gradle test'
            }
        }

        stage('Deploy & Run') {
            steps {
                // 1. 이전에 실행되던 애플리케이션 종료 (포트 8089 기준)
                bat '@for /f "tokens=5" %%a in (\'netstat -aon ^| findstr :8089 ^| findstr LISTENING\') do @taskkill /f /pid %%a'

                // 2. 새로운 애플리케이션을 백그라운드에서 실행
                // Jenkins 환경 변수를 Java 시스템 프로퍼티(-D 옵션)로 전달합니다.
                // 이렇게 하면 application.properties의 설정을 덮어쓰게 됩니다.
                bat '''
                    start "Spring Boot App" java ^
                    -Dspring.datasource.url=jdbc:mysql://%DB_HOST%/%DB_NAME% ^
                    -Dspring.datasource.username=%DB_USER% ^
                    -Dspring.datasource.password=%DB_PASS% ^
                    -jar build\\libs\\springboot-app-0.0.1-SNAPSHOT.jar
                '''
            }
        }
    }
}
