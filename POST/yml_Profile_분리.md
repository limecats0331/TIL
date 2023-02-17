```yml
spring:  
  profiles:  
    group:  
      local:
        - common  
        - local_db  
      prod:
        - common  
        - deploy_db  
---  
# common part  
spring:  
  config:  
    activate:  
      on-profile: "common"  
  
server:  
  port: 8080  
---  
spring:  
  config:  
    activate:  
      on-profile: "local_db"  
  datasource:  
    url: jdbc:h2:mem:testdb 
    username: sa  
    password:  
    driver-class-name: org.h2.Driver
---  
spring:  
  config:  
    activate:  
      on-profile: "deploy_db"  
  datasource:  
    url: jdbc:mysql://localhost:3306/local?serverTimezone=UTC&characterEncoding=UTF-8  
    username: deploy  
    password: deploy  
```

빈 등록시에도 `profile`을 지정해서 특정 `profile`에서만 등록되도록 할 수 있다.
```java
@Slf4j
@Profile("local") // 프로파일 이름 혹은 프로파일 그룹 이름이 들어감
@Component
public class Local {
    @PostConstruct
    public void execute() {
        log.info("local 프로파일 사용시만 스프링 빈으로 등록됨")
    }
    ...
}
```

이렇게 분리하면 실행할 때 로컬 환경과 배포 환경을 나눠서 실행할 수 있다.
``` terminal
java -jar -Dspring.profiles.active={group_name} {name}.jar
```