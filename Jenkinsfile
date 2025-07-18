pipeline {
    agent any

    tools {
        jdk 'JDK17'               // 등록된 이름과 일치해야 함
        gradle 'gradle-8.7'       // 등록된 이름과 일치해야 함
    }

    environment {
        DB_HOST = "${env.DB_HOST}"
        DB_NAME = "${env.DB_NAME}"
        DB_USER = "${env.DB_USER}"
        DB_PASS = "${env.DB_PASS}"
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
                bat 'gradle clean build'      // ✅ gradle 호출은 문자열 아님, 쉘 명령으로
            }
        }

        stage('Test') {
            steps {
                bat 'gradle test'
            }
        }

        stage('Run') {
            steps {
                bat 'java -jar build\\libs\\springboot-app-0.0.1-SNAPSHOT.jar'
            }
        }
    }
}