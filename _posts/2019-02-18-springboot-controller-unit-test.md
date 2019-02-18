---
layout: post
title: 'Spring Boot controller单元测试'
tags: [code]
---

- controller
```
@RestController
public class UserController {
  @RequestMapping("/getString")
    public String getString() throws ExecutionException, InterruptedException {
        return "nice";
    }
}
```
- 测试类
```
//SpringBoot1.4版本之前用的是SpringJUnit4ClassRunner.class
@RunWith(SpringRunner.class)
//SpringBoot1.4版本之前用的是@SpringApplicationConfiguration(classes = Application.class)
@SpringBootTest(classes = UserController.class)
//测试环境使用，用来表示测试环境使用的ApplicationContext将是WebApplicationContext类型的
@WebAppConfiguration
public class UserControllerTest {
    @Autowired
    private WebApplicationContext webApplicationContext;
    private MockMvc mockMvc;
    @Before
    public void setUp() throws Exception{
        //MockMvcBuilders.webAppContextSetup(WebApplicationContext context)：指定WebApplicationContext，将会从该上下文获取相应的控制器并得到相应的MockMvc；
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();//建议使用这种
    }
    @Test
    public void getString() throws Exception{
        MvcResult mvcResult=mockMvc.perform(MockMvcRequestBuilders.get("/getString")
        .accept(MediaType.TEXT_HTML_VALUE))
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
        int status=mvcResult.getResponse().getStatus();
        String content =mvcResult.getResponse().getContentAsString();
        Assert.assertEquals(200,status);
        Assert.assertEquals("nice",content);

    }

}
```
- 打印输出
```
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /getString
       Parameters = {}
          Headers = [Accept:"text/html"]
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = com.example.demo.controller.UserController
           Method = public java.lang.String com.example.demo.controller.UserController.getString() throws java.util.concurrent.ExecutionException,java.lang.InterruptedException

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"text/html;charset=ISO-8859-1", Content-Length:"4"]
     Content type = text/html;charset=ISO-8859-1
             Body = nice
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```