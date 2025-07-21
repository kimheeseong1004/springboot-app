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
                    rem 로그 파일 경로를 작업 공간 내에 설정합니다.
                    set "LOG_FILE=%WORKSPACE%\\springboot_app.log"
                    echo.
                    echo [CI] 8080 포트를 사용하는 기존 애플리케이션을 중지합니다...
                    
                    rem "usebackq" 옵션은 for 루프 안의 명령어를 쌍따옴표로 감쌀 수 있게 해줘 더 안정적입니다.
                    for /f "usebackq tokens=5" %%a in (`netstat -aon ^| findstr ":8080" ^| findstr "LISTENING"`) do (
                        echo [CI] 8080 포트에서 실행 중인 PID %%a 프로세스를 발견했습니다. 종료합니다...
                        taskkill /f /pid %%a
                    )

                    echo.
                    echo [CI] 새로운 스프링 부트 애플리케이션을 백그라운드에서 시작합니다...
                    echo [CI] 애플리케이션 로그는 %LOG_FILE% 파일에 기록됩니다.

                    rem start /B 명령어로 새 창 없이 백그라운드에서 실행합니다.
                    rem 표준 출력(stdout)과 표준 에러(stderr)를 로그 파일로 리디렉션합니다.
                    start /B "Spring Boot App" java ^
                    -Dserver.port=8080 ^
                    -Dspring.datasource.url=jdbc:mysql://%DB_HOST%/%DB_NAME% ^
                    -Dspring.datasource.username=%DB_USER% ^
                    -Dspring.datasource.password=%DB_PASS% ^
                    -jar build\\libs\\springboot-app-0.0.1-SNAPSHOT.jar > %LOG_FILE% 2>&1

                    echo.
                    echo [CI] 애플리케이션이 시작될 때까지 5초간 대기합니다...
                    timeout /t 5 /nobreak > NUL

                    echo.
                    echo [CI] 애플리케이션이 8080 포트에서 실행 중인지 확인합니다...
                    netstat -aon | findstr ":8080" | findstr "LISTENING"
                    
                    rem ERRORLEVEL이 0이면 findstr 명령이 성공한 것 (LISTENING을 찾음)
                    if %ERRORLEVEL% equ 0 (
                        echo [CI] 성공: 애플리케이션이 정상적으로 실행 중입니다.
                    ) else (
                        echo [CI] 오류: 애플리케이션 시작에 실패했습니다. 아래 로그 파일을 확인하세요.
                        echo [CI] 로그 파일의 마지막 20줄을 표시합니다:
                        echo --------------------------------------------------
                        powershell -Command "Get-Content %LOG_FILE% -Tail 20"
                        echo --------------------------------------------------
                        rem 빌드를 실패 처리합니다.
                        exit /b 1
                    )
                    
                    echo.
                    echo [CI] 애플리케이션 시작 명령이 성공적으로 전달되었습니다.
                '''
            }
        }
    }
}
