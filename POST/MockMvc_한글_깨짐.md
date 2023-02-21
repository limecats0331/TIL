# MockMvc에서 한글 깨짐 문제

MockMvc에서 테스트하다가 한글 깨짐 문제를 만났는데
여러가지 해결방법 중에 `test/resources/application.yml`를 바꾸는 방법이 제일 편하기 때문에 이를 변경해주면 된다.

```yml
server:  
  servlet:  
    encoding:  
      force-response: true
```
