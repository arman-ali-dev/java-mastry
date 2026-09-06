## 1. Testing Ki Zaroorat Kyun Hai

Ek method dekho:

```java
public int calculateFinalPrice(int price, int discountPercentage) {
    return price - (price * discountPercentage / 100);
}
```

Ye method dekhne me kaam kar raha lagta hai. Lekin **code likhna** aur **ye prove karna ki wo sahi behave kar raha hai** — ye do alag cheezein hain.

Hum manually test kar sakte hain:
```
Price = 1000
Discount = 20
Expected result = 800
```

Chalo maan lete hain ye output `800` sahi aaya. Lekin kuch sawaal abhi bhi baaki hain:
- Discount `0` ho to kya hoga?
- Discount `100` ho to kya hoga?
- Discount **negative** ho to?
- Discount `100` se zyada ho to?
- Kal ko koi is formula ko change kar de to?

**Testing basically ye verify karti hai ki code ka behaviour, requirement se match karta hai ya nahi.**

---

## Manual Testing vs Automated Testing

### Manual Testing
Developer khud:
1. Application start karta hai
2. Ek request tayaar karta hai
3. Endpoint call karta hai
4. Response dekhta hai
5. Khud decide karta hai ki result sahi hai ya nahi

Ye chhoti application ke liye chal jata hai, lekin application badhne ke saath ye **slow aur unreliable** ho jata hai.

### Automated Testing
Ek automated test:
```
Known input do → Production code chalao → Actual output aata hai → Expected output se compare karo → Pass/Fail
```

### Test Case Ke 3 Parts

```
Initial condition
Action
Expected result
```

Example:
```
Initial condition: Price ₹1000 hai, discount 20% hai
Action: Final price calculate karo
Expected result: Final price ₹800 hona chahiye
```

Testing me isi ko bolte hain **Arrange, Act, Assert**.

### Arrange, Act, Assert

**Arrange** — data aur environment tayaar karo:
```java
int price = 1000;
int discount = 20;
```

**Act** — jis behaviour ko test karna hai, use execute karo:
```java
int actualPrice = calculator.calculateFinalPrice(price, discount);
```

**Assert** — actual result ko expected result se compare karo:
```java
assertEquals(800, actualPrice);
```

Poora test aisa dikhega:
```java
@Test
void shouldApplyDiscountToPrice() {
    // Arrange
    int price = 1000;
    int discount = 20;

    // Act
    int actualPrice = calculator.calculateFinalPrice(price, discount);

    // Assert
    assertEquals(800, actualPrice);
}
```

### Tests Behaviour Verify Karte Hain, Sirf Lines Nahi

Maan lo requirement hai: *"Discount 0 aur 100 ke beech hona chahiye."*

Isse multiple behaviours nikalte hain:
- Valid discount se sahi price calculate ho
- Discount 0 se kam ho to reject ho
- Discount 100 se zyada ho to reject ho
- Negative price reject ho

Production code isse enforce karega:
```java
public int calculateFinalPrice(int price, int discountPercentage) {
    if (price < 0) {
        throw new IllegalArgumentException("Price cannot be negative");
    }
    if (discountPercentage < 0 || discountPercentage > 100) {
        throw new IllegalArgumentException("Discount must be between 0 and 100");
    }
    return price - (price * discountPercentage / 100);
}
```

**Important baat:** Tests un **behaviours** ke liye likhne chahiye jo ek method promise karta hai — sirf uske code ki har line ke liye nahi.

---

## Unit Tests aur Integration Tests

Ek common Spring Boot flow socho:
```
Controller → Service → Repository → Database
```

### Unit Test
Ek chhote se behaviour ko **isolation** me verify karta hai. Jaise `ProductService` ka test — isko in cheezon ki zaroorat nahi honi chahiye:
- Spring `ApplicationContext`
- Tomcat
- Real database
- HTTP request

Agar `ProductService`, `ProductRepository` pe depend karta hai, to real repository ki jagah ek **controlled test double** use karte hain.

### Integration Test
Ye check karta hai ki multiple **real components** ek saath sahi kaam karte hain ya nahi. Jaise: `Repository + Hibernate + H2/MySQL`.

**Asli sawaal ye nahi hai ki kaunsa annotation use kiya** — asli sawaal hai: **"is test me kitna real application infrastructure involved hai?"**

| Test type | Kya real hai? | Speed | Purpose |
|---|---|---|---|
| Unit test | Ek class/behaviour | Sabse fast | Business logic verify karna |
| Slice test | Ek application layer | Fast | Specific Spring layer verify karna |
| Integration test | Multiple components | Slow | Components saath me kaam karte hain ya nahi |
| End-to-end test | Poora user flow | Sabse slow | Complete journey verify karna |

### Testing Pyramid

Ek healthy test suite me:
```
Bahut saare fast unit tests
Kam integration tests
Bahut hi kam end-to-end tests
```

Agar **har** test Spring Boot start kare aur database se connect kare, to poora test suite slow ho jayega. Developers fir test bar-bar chalane se hichkichayenge.

Unit tests fast feedback dete hain. Integration tests confirm karte hain ki units aapas me sahi jude hue hain. **Dono zaroori hain**, kyunki dono alag problems solve karte hain.

### Kya Unit Test Karna Chahiye?

Priority do un cheezon ko jinme **behaviour ya decision** ho:
- Business calculations
- Conditional branches
- Data transformations
- Validation rules
- Exception conditions
- Authorization decisions
- Service-layer orchestration
- Important edge cases

Generally **time waste mat karo** in par:
- Plain getters/setters
- Framework ka apna behaviour
- `JpaRepository` ke built-in methods
- Aisa code jisme koi meaningful decision/transformation na ho

Ye prove karne ki zaroorat nahi ki Spring Data ka `save()` kaam karta hai — important ye hai ki application `save()` ko **sahi business conditions** ke under call kar rahi hai ya nahi.

---

## 2. JUnit Fundamentals

### JUnit Kya Deta Hai
- Test declare karne ka programming model
- `@Test` jaisi annotations
- `assertEquals()` jaise assertions
- Test discover aur execute karne ki machinery
- Maven, Gradle, IDEs ke saath integration

Modern Spring Boot tests JUnit Jupiter se aati hain: `org.junit.jupiter.api.Test`.

### Testing Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

`test` scope ka matlab hai — ye libraries sirf test compile/run karne ke liye available hain, normal production dependency ki tarah shamil nahi hoti.

`spring-boot-starter-test` deta hai: JUnit, Mockito, AssertJ, Hamcrest, Spring Test, JSON testing utilities.

### Test Directory Structure

Production code: `src/main/java`
Test code: `src/test/java`

Test package normally production package ko **mirror** karta hai:
```
src/main/java/com.example.demo.service/ProductService.java
src/test/java/com.example.demo.service/ProductServiceTest.java
```

Isse production class aur uska matching test dhoondhna aasan rehta hai.

### Pehla JUnit Test Likhna

Production class:
```java
public class PriceCalculator {
    public int calculateFinalPrice(int price, int discountPercentage) {
        if (price < 0) throw new IllegalArgumentException("Price cannot be negative");
        if (discountPercentage < 0 || discountPercentage > 100)
            throw new IllegalArgumentException("Discount must be between 0 and 100");
        return price - (price * discountPercentage / 100);
    }
}
```

Test class:
```java
class PriceCalculatorTest {

    private PriceCalculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new PriceCalculator();
    }

    @Test
    void shouldApplyDiscountToPrice() {
        int price = 1000;
        int discount = 20;
        int actualPrice = calculator.calculateFinalPrice(price, discount);
        assertEquals(800, actualPrice);
    }
}
```

**Yahan zaroori observation:** Ye test na `@SpringBootTest` use karta hai, na `@Autowired`. Ye Spring `ApplicationContext` bhi start nahi karta. `PriceCalculator` ek normal Java class hai, isliye ise seedha `new` se bana sakte hain.

Spring Boot applications, sab se pehle Java applications hi hain. Jab behaviour ko direct test kiya ja sakta ho, to **poore Spring container ko start karne ki zaroorat nahi** honi chahiye.

### @Test Samjho

`@Test` JUnit ko batata hai — is method ko discover karo aur ek test case ki tarah execute karo.

Test method names **behaviour describe** karne chahiye:

Achhe naam:
```
shouldApplyDiscountToPrice()
shouldThrowExceptionWhenDiscountIsNegative()
```

Vague naam se bacho:
```
test1()
checkMethod()
```

### Assertions

**assertEquals()**
```java
assertEquals(expectedValue, actualValue);
```

**assertTrue() / assertFalse()**
```java
assertTrue(result.isActive());
assertFalse(result.isDeleted());
```

**assertNull() / assertNotNull()**
```java
assertNull(product.getId());
assertNotNull(savedProduct.getId());
```

**assertThrows()** — jab exception khud expected behaviour ho:
```java
@Test
void shouldRejectDiscountAboveHundred() {
    IllegalArgumentException exception = assertThrows(
        IllegalArgumentException.class,
        () -> calculator.calculateFinalPrice(1000, 120)
    );

    assertEquals("Discount must be between 0 and 100", exception.getMessage());
}
```

Ye test **tabhi pass** hoga jab specified code sach me `IllegalArgumentException` throw kare. Agar koi exception nahi aayi, to test **fail** ho jayega.

### @BeforeEach aur @AfterEach

**@BeforeEach** — **har** test method se **pehle** chalta hai:
```java
@BeforeEach
void setUp() {
    calculator = new PriceCalculator();
}
```
Multiple tests ke liye shared preparation ke liye useful hai. Har test ek **predictable state** se shuru hota hai.

**@AfterEach** — har test method ke **baad** chalta hai:
```java
@AfterEach
void cleanUp() {
    // agar zaroorat ho to resources release karo
}
```
Ye tab useful hai jab test manually koi resource open karta ho jise close karna ho — sirf annotation exist karti hai isliye empty `@AfterEach` add karna zaroori nahi.

### Tests Chalana

IntelliJ se direct chala sakte ho, ya:
```
./mvnw test
```

```
Green → Expected behaviour observe hua
Red   → Expected aur actual behaviour match nahi hue
```

Ek fail hua test bhi kaam ki information deta hai — iska matlab ho sakta hai:
- Production code galat hai
- Test ki expectation galat hai
- Requirement change ho gayi hai

---

## 3. Service Layer Ko Mockito Se Unit Test Karna

### Dependency Ki Problem

```java
@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public Product getProductById(Long id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException("Product not found: " + id));
    }

    public Product createProduct(Product product) {
        boolean alreadyExists = productRepository.existsByName(product.getName());
        if (alreadyExists) throw new IllegalArgumentException("Product already exists");
        return productRepository.save(product);
    }
}
```

`ProductService` ko `ProductRepository` ke bina banaya hi nahi ja sakta:
```java
ProductService service = new ProductService(???); // ??? me kya doge?
```

Real repository use karne ke liye chahiye ho sakta hai: Spring Data proxy, `EntityManager`, Hibernate, `DataSource`, ek real database.

Agar aisa kiya, to koi failure **database ya Hibernate configuration** ki wajah se ho sakti hai, service logic ki wajah se nahi. Ye ab ek **isolated unit test** nahi rahega.

Isliye dependency ko ek **controlled test double** se replace karte hain.

### Test Doubles — 4 Types

**Dummy** — sirf isliye pass kiya jata hai kyunki method ko chahiye, test uska behaviour use nahi karta.

**Stub** — predefined answers return karta hai:
```
Jab findById(1) call ho, to ye predefined product return karo.
```

**Fake** — ek simplified lekin working implementation, jaise ek in-memory repository jo `HashMap` se bana ho, MySQL nahi.

**Mock** — ek configurable test double jo:
- Predefined values return kar sakta hai
- Predefined exceptions throw kar sakta hai
- Method calls **record** kar sakta hai
- Verify kar sakta hai ki usse kaise use kiya gaya

Mockito bina manually fake repository classes likhe mocks bana deta hai.

### Mockito Ko JUnit Ke Saath Configure Karna

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {

    @Mock
    private ProductRepository productRepository;

    @InjectMocks
    private ProductService productService;
}
```

**@ExtendWith(MockitoExtension.class)**

JUnit ko pata hai tests kaise execute karne hain, lekin ise khud Mockito annotations process karna nahi aata.

`MockitoExtension` Mockito ke lifecycle ko JUnit Jupiter se jodta hai, aur `@Mock`/`@InjectMocks` waale fields ko initialize karta hai.

**@Mock**
```java
@Mock
private ProductRepository productRepository;
```
Mockito ek object banata hai jo `ProductRepository` implement karta hai. Ye **nahi** hai:
- Spring bean
- Spring Data proxy
- Real repository
- Database se connected

Ye sirf test ke liye ek controlled replacement hai.

**@InjectMocks**
```java
@InjectMocks
private ProductService productService;
```
Mockito `ProductService` banata hai aur available mock ko uske constructor me inject kar deta hai. Conceptually ye kuch aisa karta hai:
```java
ProductRepository repository = Mockito.mock(ProductRepository.class);
ProductService productService = new ProductService(repository);
```

`@InjectMocks` ek Mockito ki convenience machinery hai — ye **Spring dependency injection nahi hai**.

### Success Scenario Test Karna

```java
@Test
void shouldReturnProductWhenProductExists() {
    // Arrange
    Product product = new Product(1L, "Laptop", 10);

    when(productRepository.findById(1L))
            .thenReturn(Optional.of(product));

    // Act
    Product result = productService.getProductById(1L);

    // Assert
    assertEquals(1L, result.getId());
    assertEquals("Laptop", result.getName());

    verify(productRepository).findById(1L);
}
```

Ye stubbing instruction:
```java
when(productRepository.findById(1L)).thenReturn(Optional.of(product));
```

Iska matlab: *"Agar is mock repository pe `findById(1L)` call hua, to ye predefined Product return karo."*

Mockito repository ko test nahi kar raha — ye ek **predictable environment** create kar raha hai jisme service ko test kiya ja sake.

### 3 Alag Concepts — Stubbing, Assertion, Verification

**Stubbing** — dependency ka behaviour define karta hai:
```java
when(repository.findById(1L)).thenReturn(Optional.of(product));
```

**Assertion** — service ka observable result check karta hai:
```java
assertEquals("Laptop", result.getName());
```

**Verification** — check karta hai ki service ne apni dependency se sahi interact kiya ya nahi:
```java
verify(repository).findById(1L);
```

```
when()   → Dependency ko control karo
assert() → Result check karo
verify() → Interaction check karo
```

`verify()` ko har mocked method ke liye call karna zaroori nahi — jab interaction khud test ho raha behaviour ka part ho, tabhi use karo.

### Exception Scenario Test Karna

```java
@Test
void shouldThrowExceptionWhenProductDoesNotExist() {
    when(productRepository.findById(99L)).thenReturn(Optional.empty());

    ProductNotFoundException exception = assertThrows(
            ProductNotFoundException.class,
            () -> productService.getProductById(99L)
    );

    assertEquals("Product not found: 99", exception.getMessage());
    verify(productRepository).findById(99L);
}
```

Repository ko instruct kiya jata hai "product absent hai" report karne ke liye. Fir test verify karta hai ki service `ProductNotFoundException` throw karta hai ya nahi.

Ye application ka **business rule** check karta hai, database ko nahi.

### Product Creation Test Karna

**Successful Creation:**
```java
@Test
void shouldCreateProductWhenNameIsUnique() {
    Product request = new Product(null, "Keyboard", 20);
    Product savedProduct = new Product(10L, "Keyboard", 20);

    when(productRepository.existsByName("Keyboard")).thenReturn(false);
    when(productRepository.save(request)).thenReturn(savedProduct);

    Product result = productService.createProduct(request);

    assertEquals(10L, result.getId());
    assertEquals("Keyboard", result.getName());

    verify(productRepository).existsByName("Keyboard");
    verify(productRepository).save(request);
}
```

**Duplicate Product:**
```java
@Test
void shouldRejectProductWhenNameAlreadyExists() {
    Product request = new Product(null, "Keyboard", 20);

    when(productRepository.existsByName("Keyboard")).thenReturn(true);

    assertThrows(IllegalArgumentException.class,
            () -> productService.createProduct(request));

    verify(productRepository).existsByName("Keyboard");
    verify(productRepository, never()).save(any(Product.class));
}
```

Ye verification bahut important hai:
```java
verify(productRepository, never()).save(any(Product.class));
```

Business rule ye hai ki **duplicate product save nahi hona chahiye**. Isliye `save()` ka **na hona** bhi expected behaviour ka part hai — aur usse explicitly verify karna zaroori hai.

### Argument Matchers

**Exact value stubbing:**
```java
when(productRepository.findById(1L)).thenReturn(Optional.of(product));
```

**Flexible stubbing:**
```java
when(productRepository.save(any(Product.class))).thenReturn(savedProduct);
```

Common matchers: `any()`, `anyLong()`, `anyString()`, `eq(value)`.

Exact value use karo jab **exact argument** matter karta ho. Matchers use karo jab test ko sirf argument ke **type/category** se matlab ho.

**Zaroori rule:** Jab ek multi-argument method me ek argument ke liye matcher use karo, to **sabhi arguments** ke liye matcher use karo:
```java
when(repository.search(eq("Laptop"), eq(true))).thenReturn(products);
```

---

## 4. Spring MVC Controllers Test Karna

### Controller Test Ko Kya Verify Karna Chahiye?

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@Valid @RequestBody Product product) {
        Product savedProduct = productService.createProduct(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedProduct);
    }
}
```

Controller **HTTP contract** ka zimmedaar hai:
- Kaunsa URL accept hota hai?
- Kaunsa HTTP method accept hota hai?
- JSON, Java me kaise convert hota hai?
- Path variables/request parameters sahi padhe jaate hain kya?
- Validation execute hoti hai kya?
- Kaunsa status code return hota hai?
- Kaunsa JSON return hota hai?
- Exceptions HTTP response me kaise convert hoti hain?
- Security request ko allow/reject karti hai kya?

Controller test ko service ke saare business-logic tests **repeat nahi karne chahiye** — wo service ke unit tests ka kaam hai.

### Controller Method Ko Direct Call Kyun Nahi Karte?

Ye likhna possible hai:
```java
ProductController controller = new ProductController(mockService);
ResponseEntity<Product> response = controller.getProduct(1L);
```

Ye Java method ko test karta hai, lekin ye Spring MVC ka behaviour verify **nahi** karta, jaise: `@GetMapping`, `@PathVariable` conversion, `@RequestBody` deserialization, `@Valid`, HTTP status handling, JSON serialization, Security filters, global exception handlers.

Isliye controller ko **Spring MVC ki request-processing infrastructure** ke through test karna chahiye.

### @WebMvcTest

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
}
```

`@WebMvcTest` ek **limited** Spring `ApplicationContext` banata hai jisme web-layer se relevant components hote hain.

```
Load hota hai:
Controller, DispatcherServlet, Jackson JSON conversion, Validation,
Controller advice, MVC configuration, Security filters

Load NAHI hota:
Real service, Real repository, Hibernate, Database, Poori application
```

Isko **test slice** bolte hain — sirf utna part load hota hai jo test ke liye zaroori ho.

### MockMvc

```java
@Autowired
private MockMvc mockMvc;
```

`MockMvc` ek **HTTP-jaisi request** create karta hai, **bina real web server start kiye**.

```
Mock HTTP request → Spring Security filter chain → DispatcherServlet → Controller → HTTP-jaisa response
```

Yahan koi `localhost`, koi port 8080, koi running Tomcat, koi Postman **nahi** hai — lekin Spring MVC apni **asli** web infrastructure se hi request process karta hai.

### @Mock vs @MockitoBean

Service unit test me humne use kiya: `@Mock` — us test me koi Spring context hi nahi tha, isliye Mockito ne bas ek normal mock field bana diya.

Controller test me, **Spring khud** `ProductController` banata hai, jiske constructor ko `ProductService` bean chahiye. Isliye mock ko **Spring ke test context ke andar** rakhna padta hai:
```java
@MockitoBean
private ProductService productService;
```

`@MockitoBean` corresponding bean ko Spring test `ApplicationContext` me create/replace karta hai, ek Mockito mock ke saath.

| Annotation | Manage kaun karta hai | Spring context me hai? |
|---|---|---|
| @Mock | Mockito | Nahi |
| @MockitoBean | Spring Test + Mockito | Haan |

Current Spring versions me `@MockitoBean` use karo. Purane Spring Boot projects me deprecated `@MockBean` mil sakta hai.

### GET Endpoint Test Karna

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockitoBean
    private ProductService productService;

    @Test
    @WithMockUser
    void shouldReturnProductWhenProductExists() throws Exception {

        Product product = new Product(1L, "Laptop", 10);
        when(productService.getProductById(1L)).thenReturn(product);

        mockMvc.perform(get("/api/products/1").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().contentTypeCompatibleWith(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("Laptop"))
                .andExpect(jsonPath("$.stock").value(10));

        verify(productService).getProductById(1L);
    }
}
```

Test bhejta hai `GET /api/products/1`. Spring MVC ko ye sab karna padta hai:
1. Request ko `@GetMapping("/{id}")` se match karna
2. URL se `1` extract karna
3. Ise `Long` me convert karna
4. Controller method call karna
5. Return hue `Product` ko JSON me serialize karna
6. HTTP response generate karna

**perform()** — mock request create aur execute karta hai.

**status()** — response status verify karta hai (`status().isOk()` = `200 OK`).

**content()** — response content JSON hai ya nahi, verify karta hai.

**jsonPath()** — response ke specific field ko check karta hai. Response `{"id": 1, "name": "Laptop", "stock": 10}` ke liye, paths hain: `$.id`, `$.name`, `$.stock`.

### POST Endpoint Test Karna

```java
@Test
@WithMockUser
void shouldCreateProductWhenRequestIsValid() throws Exception {

    Product savedProduct = new Product(10L, "Keyboard", 20);
    when(productService.createProduct(any(Product.class))).thenReturn(savedProduct);

    String requestBody = """
            {
              "name": "Keyboard",
              "stock": 20
            }
            """;

    mockMvc.perform(post("/api/products")
                    .with(csrf())
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(requestBody))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(10))
            .andExpect(jsonPath("$.name").value("Keyboard"))
            .andExpect(jsonPath("$.stock").value(20));

    verify(productService).createProduct(any(Product.class));
}
```

Ye test verify karta hai ki: `POST` mapping kaam karta hai, JSON `Product` object me convert hota hai, controller service ko call karta hai, response status `201 Created` hai, aur saved product response JSON me correctly convert hota hai.

**Important:** `POST`, `PUT`, `DELETE` jaise unsafe HTTP methods ke liye, jab CSRF protection enabled ho, `.with(csrf())` add karna padta hai — warna request CSRF ki wajah se reject ho jayegi.

### Spring Security Handle Karna

Agar Spring Security endpoint protect kar rahi hai, to ek unauthenticated request `401 Unauthorized` de sakti hai.

`@WithMockUser` use karo test ko ek **authenticated user** ki tarah run karne ke liye:
```java
@Test
@WithMockUser(username = "aditya", roles = "ADMIN")
void shouldAllowAdminToCreateProduct() {
}
```

Spring Boot 4 ke liye, ye dependency add karo:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 5. Repository Aur Integration Testing

### Repository Tests Database Kyun Use Karte Hain

Ek repository ko mock karna ye **prove nahi karta** ki real repository query kaam karti hai:
```java
ProductRepository repository = mock(ProductRepository.class);
when(repository.findByNameIgnoreCase("laptop")).thenReturn(Optional.of(product));
```

Ye sirf itna prove karta hai ki Mockito wahi value return kar raha hai jo test me configure ki gayi thi — ye **actual query kabhi execute hi nahi hoti, na validate hoti hai.**

### @DataJpaTest

```java
@DataJpaTest
class ProductRepositoryTest {
}
```

`@DataJpaTest` application ka **JPA-related slice** load karta hai.

```
Load hota hai:
Entities, Spring Data repositories, Hibernate, EntityManager, DataSource, Embedded database

Load NAHI hota:
Controllers, Normal service beans, Web server, Poori application
```

Jab embedded database present hoti hai, Spring Boot use test ke liye configure kar deta hai. Data JPA tests **transactional** hote hain, aur har test ke baad changes **rollback** ho jaate hain.

### Tests Ke Liye H2 Add Karna

H2 ek **embedded database** hai jo test process ke andar hi chalta hai.
```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

Fayde: koi alag installation nahi, koi database server start karne ki zaroorat nahi, fast test execution, har baar fresh aur isolated test environment.

### Quick Comparison

| Kya test ho raha hai | Tool/Annotation | Real components | Replace hue components | Confidence kya milta hai |
|---|---|---|---|---|
| Plain Java logic | JUnit + `@Test` | Class under test | Koi nahi | Calculations, validation, edge cases sahi hain |
| Service layer | Mockito, `@Mock`, `@InjectMocks` | Service | Repository/dependencies | Business logic aur orchestration sahi hai |
| MVC controller | `@WebMvcTest`, `MockMvc`, `@MockitoBean` | Spring MVC web layer | Service | HTTP mappings, JSON, status codes, validation, security sahi hain |
| Repository | `@DataJpaTest` + H2 | Repository, Hibernate, database | Baaki layers | Entity mappings aur queries sahi hain |

### Core Principle

**Sabse chhota test environment use karo** jo tumhare test ho rahe behaviour ko prove kar sake:
```
Plain Java behaviour        → JUnit
Service with dependencies   → JUnit + Mockito
Web-layer contract          → @WebMvcTest + MockMvc
Repository query/mapping    → @DataJpaTest + Database
```

Fast unit tests aur focused integration tests **saath me kaam karte hain** — bina test suite ko zaroorat se zyada slow banaye, reliable feedback dete hain.
