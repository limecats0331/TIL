# `Spring Security Config`
`MockMvc` 를 사용해서 테스트를 할 때, `SpringSecurity`에 관련된 `Configure`는 가져오지 않는다. 그래서 이를 수동으로 설정해주어야 한다.

```java
@ContextConfiguration(classes = {{ApplicationName}Application.class, SecurityConfig.class})
```

여러 개를 넣어야 할 때는 `{}` 안에 넣어주면 된다.

# `Service & Repository`
`MockMvc`는 `Service`와 `Repository` 를 불러오지 않기 때문에 통합 테스트로 진행하던가 혹은 `@MockBean`을 사용해서 모킹해줘야 한다.

> sample
```java
@WebMvcTest(KeywordController.class)  
@ContextConfiguration(classes = {PunpunApplication.class, SecurityConfig.class})  
class KeywordControllerTest {  
    @Autowired  
    private MockMvc mockMvc;  
    @MockBean  
    private KeywordService keywordService;  
    @MockBean  
    private KeywordRepository keywordRepository;  
  
    @Test  
    @DisplayName("모든 키워드를 가져오기 - 컨트롤러")  
    void findAllKeyword() throws Exception {  
        Keyword keyword1 = Keyword.builder()  
                .id(1L)  
                .content("test1")  
                .build();  
        Keyword keyword2 = Keyword.builder()  
                .id(2L)  
                .content("test2")  
                .build();  
        doReturn(List.of(keyword1, keyword2)).when(keywordRepository).findAll();  
        doReturn(List.of(keyword1, keyword2)).when(keywordService).findAllKeyword();  
  
        KeywordsDTO output = new KeywordsDTO(List.of("test1", "test2"));  
        String str = new Gson().toJson(output);  
  
        mockMvc.perform(get("/keywords"))  
                .andExpect(status().isOk())  
                .andExpect(content().string(str))  
                .andDo(print());  
    }  
}
```