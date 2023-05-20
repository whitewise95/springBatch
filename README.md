# 1-1 프로젝트 환경설정

##버전
- java 17
- spring boot 3.1
- gradle 7.6.1
- MySQL 8.0   


---

## 기술 스택
- JAVA SpringBoot
- JPA (Spring Data JPA)
- MySQL
- Spring Batch



---
## 1-1 - 추가된 내용
- build.gradle
```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    runtimeOnly 'com.mysql:mysql-connector-j'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    implementation 'org.springframework.boot:spring-boot-starter-batch'
}

tasks.named('test') {
    useJUnitPlatform()
}

```  

---
- yml
```yaml
server:
  port: 8090
```




