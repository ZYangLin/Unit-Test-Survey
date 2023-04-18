# Spring Boot Unit Test with JUnit 5

---

### 測試的種類

<img src="https://m.media-amazon.com/images/G/01/DeveloperBlogs/AlexaBlogs/default/pyramid._CB471590882_.png" title="source: Unit Testing: Creating Functional Alexa Skills"/>

----

- 端對端測試: 測試範圍涵蓋整個應用系統，從開始到結束的測試
- 整合測試: 測試的重點在於將各個獨立的單元結合為一個群組測試彼此間交互運作是否有誤 (不會使用mockData)
- 單元測試: 用於驗證一個函式、方法或者類別功能有無錯誤。 驗證可分為兩個面向，使用有效或無效的資料，並預期得到正確或例外處理的結果。(會使用mockData)

---

### JUnit
> Unit is a unit testing framework for the Java programming language.

- Annotations
    - @BeforeAll: 只ㄓ在第一個測試執行前會執行
    - @BeforeEach: 每個測試執行前都會執行
    - @Test: 要執行測試的方法
    - @AfterEach: 每個次測試執行後都會執行
    - @AfterAll: 只在最後一個測試執行後會執行

```java=
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class HelloTest {

    @Test
    public void oneThing() {
        // Code that tests one thing
    	System.out.println("@Test: A測試方法");
    }


    @Test
    public void anotherThing() {
        // Code that tests another thing
    	System.out.println("@Test:  B測試方法");
    }
	
    @BeforeAll
    public static void setUpClass() throws Exception {
        // Code executed before the first test method
		System.out.println("@BeforeAll");
    }

    @BeforeEach
    public void setUp() throws Exception {
        // Code executed before each test
    	System.out.println("@BeforeEach");
    }

    @AfterEach
    public void tearDown() throws Exception {
        // Code executed after each test 
    	System.out.println("@AfterEach");
    }
 
    @AfterAll
    public static void tearDownClass() throws Exception {
        // Code executed after the last test method 
    	System.out.println("@AfterAll");
    }
}
```

```java=
-----執行結果-----
@BeforeAll
    
@BeforeEach
@Test:  A測試方法
@AfterEach

@BeforeEach
@Test: B測試方法
@AfterEach

@AfterAll
```

要執行一個單元測試，只要在測試的方法上面加上 @Test 就可以執行

```java=
@Test
void testToRs() {
    EqualMortgagePaymentFxOutput equalMortgagePaymentFxOutput = new EqualMortgagePaymentFxOutput();
    equalMortgagePaymentFxOutput.setCustomerId("A156804480");
    equalMortgagePaymentFxOutput.setLoanBal(BigDecimal.valueOf(123));
    equalMortgagePaymentFxOutput.setNetLoanBal(BigDecimal.valueOf(456));
    equalMortgagePaymentFxOutput.setPayPeriod("2");
    equalMortgagePaymentFxOutput.setEndDate("25");
    equalMortgagePaymentFxOutput.setIsAllowPeriod("5");
    equalMortgagePaymentFxOutput.setNextDueDate("10");
    equalMortgagePaymentFxOutput.setNextPayableAmt(BigDecimal.valueOf(2));
    equalMortgagePaymentFxOutput.setNetBal(BigDecimal.valueOf(2));

    EqualMortgagePaymentRs rs = Mappers.getMapper(EqualMortgagePaymentMapper.class).toRs(equalMortgagePaymentFxOutput);
    log.debug(rs.toString());
}
```

----

### Mockito

> Mockito is an open source testing framework for Java.

Mockito也是一種測試框架，不同的是它可幫我們模擬原本的物件，讓我們代入自定的測試資料，並且定義回傳的結果、預期的例外 .etc

常見的 mockito 使用情境如下
- Mockito.when(object field name.method name).thenReturn(定義回傳結果)
- Mockito.when(object field name.method name).thenThrown(定義例外)
- Mockito.verify(object field name, Mockito.time(1)).method name

Mockito 更多使用方法
- Mockito.when(物件方法名稱).thenReturn(定義的結果)
    - Mockito.when(employeeDao.getEmployeeById(2)).thenReturn(new User(200, "I'm mocker"));
        - Employee employee = employeeService.getEmployeeById(2);
        - Assert.assertEquals(employee.getId(), new Integer(200));
        - Assert.assertEquals(employee.getName(), "I'm mocker")
        
    - 接收任何整數 Mockito.when(employeeService.getEmployeeById(Mockito.anyInt())).thenReturn(new Employee(5, "I'm mocker"));
        - Employee employee1 = employeeService.getEmployeeById(9); //回傳 I'm mocker
        - Employee employee2 = employeeService.getEmployeeById(13); //回傳 I'm mocker
        
    - 接收特定數值 Mockito.when(employeeService.getEmployee(7)).thenReturn(new Employee(7, "I'm mocker"));
        - Employee employee1 = employeeService.getEmployeeById(7); //回傳 I'm mocker
        - Employee employee2 = employeeService.getEmployeeById(13);
        
    - 接收任何類別 Mockito.when(employeeService.insertEmployee(Mockito.any(Employee.class))).thenReturn(100);
        - Integer i = employeeService.insertEmployee(new Employee()); // 回傳整數 100

----

- Mockito thenThrow
    - Mockito.when(employeeService.getEmployeeById(6)),thenThrow(new RuntimeException("mock throw excpetion"));
        - Employee employee = employeeService.getEmployeeById(6); // 會拋出例外
        
    - Mockito.doThrow(new RuntimeException("mock throw excpetion")).when(employeeSercie).print();
        - employeeService.print(); // 會拋出例外

----

- Mockito verify 
    - 檢查呼叫 employeeService getEmployeeById() 方法1次且參數為7
        - Mockito.verify(employeeService, Mockito.times(1)).getEmployeeById(7);
        
    - 驗證呼叫順序，並且第一次參數是2、第二次參數是5、最後才呼叫 insertEmployee()方法
        - InOrder inOrder = Mockito.inOrder(employeeService);
        - inOrder.verify(employeeService).getEmployeeById(2);
        - inOrder.verify(employeeService).getEmployeeById(5);
        - inOrder.verify(employeeService).insertEmployee(Mockito.any(Employee.class));

---

#### Demo1

```java=
// 引入Spring test context with JUnit 5，並搭配 @MockBean 將真正的 java bean 透過模擬的方式產生在IOC container
@ExtendWith(SpringExtension.class)
// 導入所有在 @Configuration 中定義的 bean
@Import(EmployeeServiceConfiguration.class)
public class EmployeeServiceTest {

    @Autowired
    private EmployeeService employeeService;

//	@TestConfiguration
//	static class EmployeeServiceConfiguration {
//		@Bean
//		public EmployeeService employeeService() {
//			return new EmployeeService();
//		}
//	}

    // 將真正的 Java Bean 透過模擬的方式產生在 IOC Container
    // 在此 mockito 會幫我們模擬出 bean 讓我們調用
    @MockBean
    private EmployeeRepository employeeRepository;

    @BeforeAll
    public static void beforeAll() {
    	System.err.println("準備運行測試方法");
    }

    @Test
    public void testGetEmployeeByName() {
    	System.err.println("測試結果為");
		
        // 當 employeeRepository.getEmployeeByName("Ted") 被執行時，回傳 new Employee("Mark")
        Mockito.when(employeeRepository.getEmployeeByName("Ted")).thenReturn(new Employee("Mark"));
        // 執行 employeeRepository.getEmployeeByName("Ted")
        Employee employee = this.employeeService.getEmployeeByName("Ted");
		
        // 檢核是否有呼叫 employeeRepository.getEmployeeByName 方法一次並帶入參數為 "Ted" 若不符合則噴出錯誤
        Mockito.verify(this.employeeRepository, times(1)).getEmployeeByName(Mockito.eq("Ted"));
		
        Assertions.assertNotNull(employee);
        Assertions.assertEquals("Mark", employee.getName());
    }
	
    @AfterAll
    public static void afterAll() {
        System.err.println("結束運行測試方法");
    }
}
```

```java
@TestConfiguration
public class EmployeeServiceConfiguration {
	
    @Bean
    public EmployeeService EmployeeService() {
        return new EmployeeService();
    }
}
```

```java
@Service
public class EmployeeService {
	
    @Autowired
    private EmployeeRepository employeeRepository;
	
    public Employee getEmployeeByName(String name) {
    	return employeeRepository.getEmployeeByName(name);
    }
}
```

```java
@Repository
public class EmployeeRepository {
	
    public Employee getEmployeeByName(String name) {
    	return null;
    }
}
```


**由於我們分了好幾層(像是 controller, service, repository .etc)， 且裡面有很多使用autowired的依賴注入，所以若是 @Mockbean 或 @Import 沒寫好，以下這兩個exception就很容易出現**
1. org.springframework.beans.factory.UnsatisfiedDependencyException
2. org.springframework.beans.factory.NoSuchBeanDefinitionException


---

#### 哪些類型無需做單元測試

- what not to test
    - 不含邏輯的不測(但類別內有dependency的則要測驗是否能被invoked)

1. interface 系列

2. Rq, Rs beans
   
3. Enums status code
    
4. Mapper 
    - 單純input to output 的轉換，沒特別使用 @Mapping 指定對照的欄位，也無使用其他的方法或撰寫邏輯在裡面

參考資料 - 什麼時候不用寫測試

1. [Unit Testing - What not to test](https://stackoverflow.com/questions/1316848/unit-testing-what-not-to-test)
2. [When is it appropriate to not unit test?](https://softwareengineering.stackexchange.com/questions/66480/when-is-it-appropriate-to-not-unit-test)
3. [When is unit testing inappropriate or unnecessary? ](https://softwareengineering.stackexchange.com/questions/147055/when-is-unit-testing-inappropriate-or-unnecessary)


---

### Unit Test in Controller, Service, Repository

Controller
~ 確認可以處理我們發送的request並回覆正確的結構

Controller 要負責的項目
1. Verify Http Request Matching
    ```java=
    mockMvc.perform(post("/forums/42/register")
        .contentType("application/json"))
        .andExpect(status().isOk());
    ```
2. Verify Input Deserialization
    ```java=
    mockMvc.perform(post("/forums/{forumId}/register", 42L)
            .contentType("application/json")
            .param("sendWelcomeMail", "true")
            .content(objectMapper.writeValueAsString(user)))
            .andExpect(status().isOk());
    ```

3. Verify Input Validation
    ```java=
    @Value
    public class UserResource {

      @NotNull
      private final String name;

      @NotNull
      private final String email;

    }

    @Test
    void whenNullValue_thenReturns400() throws Exception {
      UserResource user = new UserResource(null, "zaphod@galaxy.net");

      mockMvc.perform(post("/forums/{forumId}/register", 42L)
          ...
          .content(objectMapper.writeValueAsString(user)))
          .andExpect(status().isBadRequest());
    }
    ```

4. Verify Output Serializtion
    ```java=
    @Test
    void whenValidInput_thenReturnsUserResource() throws Exception {
      MvcResult mvcResult = mockMvc.perform(...)
          ...
          .andReturn();

      UserResource expectedResponseBody = ...;
      String actualResponseBody = mvcResult.getResponse().getContentAsString();

      assertThat(actualResponseBody).isEqualToIgnoringWhitespace(
                  objectMapper.writeValueAsString(expectedResponseBody));
    }
    ```

參考資料 - Testing in Controller, service, repository

1. [Testing in Spring Boot](https://www.baeldung.com/spring-boot-testing)
2. [Spring Boot - Rest Controller Unit Test](https://www.tutorialspoint.com/spring_boot/spring_boot_rest_controller_unit_test.htm)
3. [How to test services, endpoints, and repositories in Spring Boot](https://medium.com/free-code-camp/unit-testing-services-endpoints-and-repositories-in-spring-boot-4b7d9dc2b772)
4. [The “Testing with Spring Boot” Series](https://reflectoring.io/spring-boot-test/)
5. [Testing MVC Web Controllers with Spring Boot and @WebMvcTest](https://reflectoring.io/spring-boot-web-controller-test/)


Service
~ 確認業務邏輯是按照預設的正確運行

Repository
~ 確認我們對資料表制定的規範有正確的實施
  ```java=
  @RunWith(SpringRunner.class)
  @DataJpaTest
  public class UserRepositoryTest {
      @Autowired
      TestEntityManager entityManager;

      @Autowired
      UserRepository sut;

      @Test
      public void it_should_save_user() {
          User user = new User();
          user.setName("test user");

          user = entityManager.persistAndFlush(user);

          assertThat(sut.findById(user.getId()).get()).isEqualTo(user);
      }
  }
  ```

---

>  撰寫測試案例的好處如下:
1. 幫助改善軟體品質、確保系統的穩定度、減少未來軟體的維護成本
2. 幫助驗證軟體是否滿足終端使用者需求
3. 單元測試是種`living documentation`。 開發者可以參考測試而對模塊的邏輯和整個系統有基本的了解
4. 讓測試者徹底思考盡可能以不同的角度進行測試
5. 測試案例是可重用的，且可被任何人執行
6. 幫助我們更了解程式的重構

- 測試案例包含的面向
    - **目的**: 測試人員預期達到的目標。 測試案例撰寫者需具有end to end的商業邏輯。
    - **先決條件**: 任何測試執行時所需的dependency, initialization 等先決條件必須先列出
    - **測試案例步驟**: 測試步驟為實現測試目標所需遵循的步驟。測試步驟要包含測試所需的必要數據和資訊
    - **測試數據**
    - **預期結果和期望結果**

---

##### deeper - 流程
- @ExtendWith(SpringExtension.class) 引入 testContext
    - SpringExtension 職責為管理 test 生命週期
        - 管理 TestContext(new one or get cache context)
        - test class constructor or test method dependency injection
        - cleanup and housekeeping tasks after the test
    - TestContextManager: 管理單個TestContext，以及在定義的執行點像listner發送event
        ```java
        public class SpringExtension {
            @Override
            public void beforeAll(ExtensionContext context) throws Exception {
                getTestContextManager(context).beforeTestClass();
            }
        }     
        ```
        - TestContext: 隨著測試的進行，Manager會更新 TestContext
        - TestExecutionListner: 對TestContextManager發送的訊號做出反應
        - SmartContextLoader: 如有需要 TestContext 會委託加載applicationContext

---

##### Unit test 參考資料
1. [What the Heck Is the SpringExtension Used For?](https://rieckpil.de/what-the-heck-is-the-springextension-used-for/)
2. [mockito-verify cookbook](https://www.baeldung.com/mockito-verify)
3. [Spring Testing(官方文件)](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing-introduction)
4. [Controller, Service, and Repository Layer Unit Testing using JUnit and Mockito](https://blog.devgenius.io/spring-boot-deep-dive-on-unit-testing-92bbdf549594)
