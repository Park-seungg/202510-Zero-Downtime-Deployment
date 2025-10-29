# 첫 번째 스테이지: 빌드 스테이지
FROM gradle:jdk-21-and-23-graal-jammy AS builder

# 작업 디렉토리 설정
WORKDIR /app

# 소스 코드와 Gradle 래퍼 복사
COPY build.gradle.kts .
COPY settings.gradle.kts .

# gradle wrapper와 wrapper 디렉토리 복사 (wrapper가 존재하면 사용)
COPY gradlew .
COPY gradle/ gradle/

# gradlew 실행 권한 부여
RUN chmod +x ./gradlew

# 종속성 설치 (gradle 대신 gradlew 사용)
RUN ./gradlew dependencies --no-daemon --stacktrace

# 소스 코드 복사
COPY .env .
COPY src src

# 애플리케이션 빌드 (테스트는 CI 파이프라인에서 별도로 실행하거나 필요시 제거)
RUN ./gradlew build --no-daemon --stacktrace -x test

# 두 번째 스테이지: 실행 스테이지
FROM container-registry.oracle.com/graalvm/jdk:21

# 작업 디렉토리 설정
WORKDIR /app

# 첫 번째 스테이지에서 빌드된 JAR 파일 복사
COPY --from=builder /app/build/libs/*.jar app.jar
COPY --from=builder /app/.env .env

# 실행할 JAR 파일 지정
ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-jar", "app.jar"]