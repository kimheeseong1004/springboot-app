pipeline {
    agent any

    tools {
        // Jenkins 설정과 일치하는 JDK, Gradle 이름 사용
        jdk 'JDK17'
        gradle 'gradle-8.7' 
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
                // 단일의 안정적인 배치 스크립트로 통합하고, 로깅을 강화합니다.
                bat '''
                    @echo off
                    set "LOG_FILE=%WORKSPACE%\\springboot_app.log"
                    echo.
                    echo [CI] Stopping any existing application on port 8080...
                    
                    for /f "usebackq tokens=5" %%a in (`netstat -aon ^| findstr ":8080" ^| findstr "LISTENING"`) do (
                        echo [CI] Found process with PID %%a listening on port 8080. Terminating...
                        taskkill /f /pid %%a
                    )

                    echo.
                    echo [CI] Starting new Spring Boot application in the background...
                    echo [CI] Application log will be written to: %LOG_FILE%

                    start /B "Spring Boot App" java ^
                    -Dserver.port=8080 ^
                    -Dspring.datasource.url=jdbc:mysql://%DB_HOST%/%DB_NAME% ^
                    -Dspring.datasource.username=%DB_USER% ^
                    -Dspring.datasource.password=%DB_PASS% ^
                    -jar build\\libs\\springboot-app-0.0.1-SNAPSHOT.jar > %LOG_FILE% 2>&1

                    echo.
                    echo [CI] Waiting for 5 seconds for the application to start...
                    rem Using ping for a reliable delay instead of timeout
                    ping -n 6 127.0.0.1 > NUL

                    echo.
                    echo [CI] Checking if the application is running on port 8080...
                    netstat -aon | findstr ":8080" | findstr "LISTENING"
                    
                    if %ERRORLEVEL% equ 0 (
                        echo [CI] SUCCESS: Application is running correctly.
                    ) else (
                        echo [CI] ERROR: Application failed to start. Please check the log file below.
                        echo [CI] Displaying the last 20 lines of the log file:
                        echo --------------------------------------------------
                        powershell -Command "Get-Content -Path '%LOG_FILE%' -Tail 20 -ErrorAction SilentlyContinue"
                        echo --------------------------------------------------
                        exit /b 1
                    )
                    
                    echo.
                    echo [CI] Application start command has been issued successfully.
                '''
            }
        }
    }
}
