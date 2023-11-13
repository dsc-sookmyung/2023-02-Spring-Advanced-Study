- 테스트: 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해서 만든 코드에 대한 확신을 주는 작업 → 결함 제거 가능
- 작은 단위로 쪼개서 집중하는 것이 좋음 (관심사의 분리)
- 단위 테스트: 충분히 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위의 단위 (작을수록 좋음)
- 개발자가 코드를 스스로 확인하기 위해 사용 → 개발자 테스트, 프로그래머 테스트
- 자동 테스트: 자주 반복 가능

### UserDaoTest의 문제점

---

```java
 User user2 = userDao.get(user.getId());
 System.out.println(user2.getName());
 System.out.println(user2.getPassword());
```

</br>

- 수동 확인 작접의 번거로움: 테스트의 결과를 확인하는 일도 사람의 책임, 실수할 가능성 있음
- 실행 작업의 번거로움: main 메소드를 여러 번 실행해야 함

</br>

# 1️⃣ UserDatoTest 개선

## 테스트 검증의 자동화

- 테스트 성공
- 테스트 실패
  1. 테스트 에러
  2. 테스트 실패
- 코드의 동작에 영향을 미칠 수 있는 변화가 생기면 다시 실행 가능

```java
if (!user.getName().equals(user2.getName())){
    System.out.printf("테스트 실패 (name)");
} else if(!user.getPassword().equals(user2.getPassword())){
    System.out.printf("테스트 실패 (password)");
} else{
		System.out.printf("조회 테스트 성공");
}
```

## 테스트의 효율적인 수행과 결과 관리

- main() 메소드를 이용한 테스트는 규모가 커질 때 테스트 수행에 부담이 됨
  - 제어권을 직접 갖음

### JUnit으로 검증 코드 전환

- assertThat(): 첫 번째 파라미터의 값을 뒤에 나오는 매처라고 불리는 조건과 비교
  - 일치: 다음으로 넘어감
  - 실패: 테스트 실패

```java
assertThat(user2.getName(), is(user.getName()));
assertThat(user2.getPassword(), is(user.getPassword()));
```

</br>

# 2️⃣ 개발자를 위한 테스트 프레임워크 JUnit

- 스프링 테스트 모듈도 JUnit 사용
- 단순하고, 테스트 작성 시 필요한 여러 가지 부가기능 제공
- 자바 IDE는 JUnit 테스트 지원 기능을 내장하고 있음

## 테스트 결과의 일관성

- 테스트가 외부 상황에 따라 성공하기도 하고 실패하기도 함
- 코드에 변경 사항이 없다면 테스트는 항상 동일한 결과를 내야함

### deleteAll()의 getCount() 추가

---

- addAndGet() 테스트에 deleteAll()과 getCount() 추가

```java
public void deleteAll() throws SQLException {
    Connection c = dataSource.getConnection();
    PreparedStatement ps = c.prepareStatement("delete from users");

    ps.executeUpdate();

    ps.close();
    c.close();
}
```

```java
public int getCount() throws SQLException {
    Connection c = dataSource.getConnection();
    PreparedStatement ps = c.prepareStatement("select count(*) from users");

    ResultSet rs = ps.executeQuery();
    rs.next();
    int count = rs.getInt(1);

    rs.close();
    ps.close();
    c.close();

    return count;
}
```

```java
@Test
public void addAndGet() throws SQLException {

		...
		// 실행 전에 deleteAll()
    userDao.deleteAll();

		// getCount() 기능 검증
    assertEquals(userDao.getCount(), 0);
 }
```

## 포괄적인 테스트

- JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않음
- 모든 테스트는 실행 순서에 상관없이 독립적으로 항상 동일한 결과를 출력해야 함

### addAndGet() 테스트 보완

---

- 주어진 id에 해당하는 정확한 User 정보인지 확인

```java
assertEquals(user1.getId(), user.getId());
assertEquals(user1.getName(), user.getName());
assertEquals(user1.getPassword(), user.getPassword());
```

### get() 예외조건에 대한 테스트

---

- 정상적으로 테스트를 마치면 실패
- 예외가 반드시 발생해야 하는 경우를 테스트할 때 유용

```java
@Test(exptected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicaionContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);

    //모든 데이터를 삭제하고 이를 검증하기 위해 추가되었다.
    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

		dao.get("unknown_id");
}
```

```java
public User get(String id) throws SQLException {
    ...
    ResultSet rs = ps.executeQuery();

    User user = null; //user를 null로 초기화
    if(rs.next()) {
        user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));
    }

    rs.close();
    ps.close();
    c.close();

		// user 없으면 에러
    if(user == null) { throw new EmptyResultDataAccessException(1); }

    return user;
}
```

## 테스트가 이끄는 개발

### 기능설계를 위한 테스트

---

|      | 단계               | 내용                                        | 코드                                                   |
| ---- | ------------------ | ------------------------------------------- | ------------------------------------------------------ |
| 조건 | 어떤 조건을 가지고 | 가져오는 사용자 정보가 존재하지 않는 경우에 | dao.deleteAll(); assertThat(dao.getCount(),is(0));     |
| 행위 | 무엇을 할때        | 존재하지 않는 id로 get() 을 실행하면        | get("unknown_id")                                      |
| 결과 | 어떤 결과가 나온다 | 특별한 예외가 던져진다.                     | @Test(expected = EmptyResultDataAccessException.class) |

- 기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 테스트 코드가 일부분 담당하고 있음
- 코드로 된 설계 문서
- 설계한 대로 코드가 동작하는지 빠르게 검증 가능

### 테스트 주도 개발

---

- 만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발
- 테스트 우선 개발로도 불림
- “실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다.”
  - 단위테스트를 만들 수 있음
- 코드를 만들어 테스트를 실행하는 사이의 간격이 매우 짧음
  - 오류를 빨리 찾아낼 수 있음

## 테스트 코드 개선

### @Before

---

- JUnit이 제공하는 애노테이션, @Test 메소드가 실행되기 전에 먼저 실행돼야 하는 메소드를 정의
- 테스트 메소드 개수만큼 반복

```java
public class UserDaoTestt {
		private UserDao dao; // setUp() 메소드에서 만드는 오브젝트를 테스트 메소드에서 사용할 수 있도록 인스턴스 변수로 선언

		@BeforeEach // @Before와 동일
    public void setUp(){
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		    dao = context.getBean("userDao", UserDao.class);
    }
		...
}
```

- 테스트 수행 방식
  1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다
  2. 테스트 클래스의 오브젝트를 하나 만든다.
  3. @Before가 붙은 메소드가 있으면 실행한다
  4. @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
  5. @After가 붙은 메소드가 있으면 실행한다.
  6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
  7. 모든 테스트의 결과를 종합해서 돌려준다.
- 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만듦
  → 각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 보장
  ![JUnit의 테스트 메소드 실행 방법](https://prod-files-secure.s3.us-west-2.amazonaws.com/cffbc2f7-d0a1-4cb5-bca1-2978cc0c8da7/fab2a5e4-c241-4556-a41c-4dbaf52137a7/Untitled.png)
  JUnit의 테스트 메소드 실행 방법

### 픽스처

---

- 테스트를 수행하는 데 필요한 정보나 오브젝트
  ex) UserDaoTest의 dao, 테스트 중 add() 메소드에 전달하는 User 오브젝트
- 여러 테스트에서 반복적으로 사용 → @Before 메소드 이용

```java
public class UserDaoTest {
		private UserDao dao;
		private User user1;
		private User user2;

		@Before
		public void setUp() {
				this.user1 = new User("goeun", "고은", "springno1");
				this.user2 = new User("pochak", "포착", "springno2");
		}
}
```

</br>

# 3️⃣ 스프링 테스트 적용

- 애플리케이션 컨텍스트가 만들어질 때 모든 싱글톤 빈 오브젝트를 초기화
- 어떤 빈은 오브젝트가 생성될 때 자체적인 초기와 작업을 진행 → 많은 시간 필요
- 애플리케이션 컨텍스트 초기화 시 독자적을으로 많은 리소스를 할당하거나 독립적인 스레드를 띄우는 빈도 있음
- 애플리케이션 컨텍스트 내의 빈이 할당한 리소스를 정리하지 않으면 다음 테스트에서 문제가 발생할 수 있음
- 애플리케이션 컨텍스트처럼 생성에 많은 시간과 자원이 소모될 경우 테스트 전체가 공유하는 오브젝트를 만들기도 함
  - 빈은 싱글톤으로 만들어져서 상태를 갖지 않으므로 공유해도 괜찮음
- JUnit이 매번 테스트 클래스의 오브젝트를 새로 만들어서 여러 테스트가 함께 참조할 애플리케이션 컨텍스트를 오브젝트 레벨에 저장해두면 안됨
  → 스태틱 필드에 저장 (@BeforeClass 스태틱 메소드: 테스트 클래스 전체에 걸쳐 한 번만 실행되는 메소드로 JUnit이 지원)
  → 대신 스프링이 직접 제공하는 애플리케이션 컨텍스트 테스트 지원 기능 사용이 더 편리

## 테스트를 위한 애플리케이션 컨텍스트 관리

- JUnit의 테스트 컨텍스트 프레임워크: 테스트에서 필요로 하는 애플리케이션 컨텍스트를 만들어서 모든 테스트가 공유 가능

### 스프링 테스트 컨텍스트 프레임워크 적용

---

```java
@RunWith(SpringJUnit4ClassRunner.class) // 스프링 테스트 컨텍스트 프레임워크의 JUnit 확장 기능 지정
@ContextConfiguration(locations = "/applicationContext.xml") // 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 설정 파일 위치 지정
public class UserDaoTest {
    @Autowired
    private ApplicationContext context; // 테스트 오브젝트가 만들어지고 나면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입됨
```

### 테스트 메소드의 컨텍스트 공유

---

- JUnit 확장 기능은 테스트가 실행되기 전에 애플리케이션 컨텍스트를 한 번만 만들어두고 테스트 오브젝트가 만들어질 때마다 특별한 방법을 이용해 애플리케이션 컨텍스트 자신을 테스트 오브젝트의 특정 필드에 주입
  - 일종의 DI, 애플리케이션 오브젝트 사이의 관계 관리를 위한 DI와 다름)
  - **애플리케이션 컨텍스트를 공유**하여 사용하는 것임

### 테스트 클래스의 컨텍스트 공유

---

- 여러 개의 테스트 클래스가 모두 같은 설정 파일을 가진 애플리케이션 컨텍스트를 사용하면 **테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유**하게 해줌

```java
// 두 테스트 클래스의 모든 메소드가 하나의 애플리케이션 컨텍스트 공유
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
public class UserDaoTest {..}

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "/applicationContext.xml")
public class UserDaoTest {..}
```

### @Autowired

---

- 테스트 컨텍스트 프레임워크는 @Autowired가 붙은 인스턴스 변수 타입과 일치하는 컨텍스트 내의 빈을 찾음 → 일치하는 빈이 있으면 인스턴스 변수에 주입
  - 생성자, 수정자 메소드가 없어도 주입 가능
  - 타입에 의한 자동와이어링: 별도의 DI 설정 없이 필드의 타입 정보를 이용해 빈을 자동으로 가져옴
- 스프링 애플리케이션 컨텍스트는 초기화할 때 자기 자신도 빈으로 등록
  - 애플리케이션 컨텍스트에 ApplicationContext 타입의 빈이 존재, 따라서 DI도 가능
- @Autowired를 이용해 애플리케이션 컨텍스트가 갖고 있는 빈을 DI 받을 수 있으므로 getBean() 대신 UserDao를 빈으로 직접 DI 받을 수 있음
  ```java
  public class UserDaoTest {
  		@Autowired
  		UserDao dao; // UserDao 타입 빈을 직접 DI 받음
  }
  ```
- 변수에 할당 가능한 타입을 가진 빈을 자동으로 찾음
  - SimpleDriverDataSource 클래스 타입, DataSource 타입(인터페이스)으로 변수 선언해도 됨
  - 빈 하나를 선택할 수 없는 경우에는 변수의 이름과 같은 이름의 빈을 확인
    → 변수 이름으로도 찾을 수 없으면 예외 발생
- SimpleDriverDataSource, DataSource 타입으로의 선언은 테스트에서 빈을 어떤 용도로 사용하는지에 따라 다름
  - DataSource: 단순히 DataSource에 정의된 메소드를 테스트에서 사용하고 싶은 경우, dataSource 빈의 구현 클래스를 변경하더라도 테스트 코드를 수정할 필요가 없음
  - SimpleDriverDataSource: 해당 타입의 오브젝트 자체에 관심이 있는 경우, XML 프로퍼티로 설정한 DB 연결정보 확인, SimpleDriverDataSource 클래스의 메소드를 직접 이용해서 테스트해야 하는 경우
    → 코드 내부구조와 설정 등을 알고 의도적으로 내용을 검증해야 할 필요가 없다면 가능한 한 인터페이스를 사용해서 애플리케이션 코드와 느슨하게 연결해두는 편이 좋음

### DI와 테스트

---

- SimpleDriverDataSource만 사용한다고 하더라도 인터페이스를 두고 DI를 적용해야 함

1. 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없음
2. 클래스의 구현 방식은 바뀌지 않는다고 하더라도 인터페이스를 두고 DI를 적용하게 해두면 다른 차원의 서비스 기능을 도입할 수 있음 (DB 커넥션의 개수 카운팅)
3. 효율적인 테스트를 위해 DI 사용이 필요함 (작은 단위의 대상에 대해 독립적으로 만들어지고 실행되게 하는 데 중요)

### 테스트 코드에 의한 DI

---

- 여러 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야할 경우라면 스프링의 설정을 이용한 DI 방식의 테스트를 이용하면 편리 (수동 DI)
- 애플리케이션 컨텍스트에서 applicationContext.xml 파일의 설정 정보를 따라 구성한 오브젝트를 가져와 의존관계를 강제로 변경한 것이므로 주의해서 사용해야 함
  → @DirtiesContext 사용
- @DiritesContext: 해당 애노테이션이 붙은 테스트 클래스에는 애플리케이션 컨텍스트 공유를 허용하지 않음, 테스트 메소드 수행 후 매번 새로운 애플리케이션 컨텍스트를 다음 테스트가 사용할 수 있게 함
  - 메소드 레벨에도 사용 가능

```java
@DirtiesContext // 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경함을 테스트 컨텍스트 프레임워크에 알려줌
public class UserDaoTest {
		@Autowired
		UserDao dao;

		//...

		@BeforeEach
		public void setUp() {
				//...
		    DataSource = new SingleConnectionDataSource( // UserDao가 사용할 DataSource 오브젝트 직접 생성
						"jdbc:postgresql://localhost/test", "postgres", "password", true
				 );
		    dao.setDataSource(dataSource); // 코드에 의한 수동 DI
}
```

### 컨테이너 없는 DI 테스트

---

- UserDao, DataSource 구현 클래스에는 스프링의 API를 직접 사용하거나 애플리케이션 컨텍스트를 이용하는 코드가 없음
- 스프링 DI 컨테이너에 의존하지 않으므로 테스트 코드에서 직접 오브젝트를 만들고 DI 해서 사용해도 됨
- 장점: 테스트 시간 절약 가능, 단순하고 이해하기 쉬움
- 단점: 매번 새로운 UserDao 오브젝트가 만들어짐

```java
public class UserDaoTest {
  UserDao dao;

  //...

  @BeforeEach
  public void setUp() {
    //...
    dao = new UserDao();
    DataSource = new SingleConnectionDataSource(
      "jdbc:postgresql://localhost/test", "postgres", "password", true
    );
    dao.setDataSource(dataSource); // 오브젝트 생성, 관계설정 등을 모두 직접 해줌
  }
```

- DI 컨테이너나 프레임워크는 DI를 편하게 적용하도록 도움을 주는 것
  - 컨테이너가 DI를 가능하게 하는 것이 아님

<aside>
📌 **침투적 기술과 비침투적 기술**

---

- 침투적 기술: 기술 적용시 애플리케이션 코드에 기술 관련 API가 등장하거나, 특정 인터페이스나 클래스를 사용하도록 강제하는 기술
  → 애플리케이션 코드가 해당 기술에 종속
- 비침투적 기술: 애플리케이션 로직을 담은 코드에 영향을 주지 않고 적용 가능 (스프링)
</aside>

### DI를 이용한 테스트 방법 선택

---

1. 스프링 컨테이너 없이 테스트
   - 테스트 수행속도가 빠름
   - 테스트 자체가 간단함
   - 오브젝트 생성과 초기화가 단순하면 사용
2. 스프링의 설정을 이용한 DI 방식의 테스트
   - 여러 오브젝트들의 복잡한 의존관계일 경우
   - 환경에 따라 각기 다른 설정파일을 구성 (개발, 테스트, 운영)
3. @DirtiesContext 붙인 수동 DI 테스트
   - 예외적인 의존관계를 강제로 구성해서 테스트할 경우

</br>

# 4️⃣ 학습 테스트로 배우는 스프링

- 학습 테스트: 자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트 작성
- 목적: 자신이 사용할 API나 프레임워크의 기능을 테스트로 보면서 사용 방법을 익힘 (이해, 검증)

## 학습 테스트의 장점

- 다양한 조건에 따른 기능을 손쉽게 확인 가능
- 학습 테스트 코드를 개발 중에 참고 가능
- 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와줌\
- 테스트 작성 훈련
- 기술 공부의 즐거움

## 학습 테스트 예제

### JUnit 테스트 오브젝트 테스트

---

```java
public class JUnitTest {
    static JUnitTest testObject;

    @BeforeAll
    public static void beforeAll() {
        testObject = new JUnitTest();
    }

    @AfterEach
    public void afterEach() {
        testObject = this;
    }

    @Test
    public void test1() {
        assertNotSame(testObject, this);
        System.out.println("testObject = " + testObject);
        System.out.println("this = " + this);
    }

    @Test
    public void test2() {
        assertNotSame(testObject, this);
        System.out.println("testObject = " + testObject);
        System.out.println("this = " + this);
    }

    @Test
    public void test3() {
        assertNotSame(testObject, this);
        System.out.println("testObject = " + testObject);
        System.out.println("this = " + this);
    }
}
```

### 스프링 테스트 컨텍스트 테스트

---

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/junit.xml")
public class JUnitTest {
		// 테스트 컨텍스트가 매번 주입해주는 애플리케이션 컨텍스트는 같은 오브젝트인지 테스트로 확인
    @Autowired
    ApplicationContext context;

    static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
    static ApplicationContext contextObject = null;

    @Test
    public void test1() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
        assertThat(contextObject == null || contextObject == this.context, is(true));
        contextObject = this.context;
    }

    @Test
    public void test2() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
        assertThat(contextObject == null || contextObject == this.context, is(true));
        contextObject = this.context;
    }

    @Test
    public void test3() {
        assertThat(testObjects, not(hasItem(this)));
        testObjects.add(this);
        assertThat(contextObject, either(is(nullValue())).or(is(this.context)));
        contextObject = this.context;
    }

}
```

## 버그 테스트

- 코드에 오류가 있을 때 오류를 가장 잘 드러낼 수 있는 테스트
- 실패하도록 만들어야 함

1. 테스트의 완성도를 높여줌
2. 버그의 내용을 명확하게 분석하게 해줌
3. 기술적인 문제를 해결하는 데 도움이 됨

</br>

<aside>
📌 동등 분할

---

- 같은 결과를 내는 값의 범위를 구분해서 각 대표 값으로 테스트를 하는 방법 (ex. true, false, 예외 발생)
</aside>

</br>

<aside>
📌 경계값 분석

---

- 에러는 동등분할 범위의 경계에서 주로 많이 발생한다는 특징을 이용
- 경계의 근처에 있는 값을 이용해 테스트하는 방법
- 숫자의 입력 값인 경우 0이나 그 주변 값 또는 정수의 최대값, 최소값 등
</aside>

</br>

# 5️⃣ 정리

- 테스트는 자동화돼야하고 빠르게 실행 가능해야 함
- main() 대신 JUnit 프레임워크 이용한 테스트가 더 편리
- 테스트 결과는 일관성 필요
- 테스트는 포괄적으로 작성 (충분한 검증)
- 코드 작성과 테스트 수행의 간격이 짧을수록 효과적
- 테스트하기 쉬운 코드가 좋은 코드
- TDD 유용
- 테스트 코드도 적절한 리팩토링 필요
- 테스트 메소드들의 공통 준비 작업과 정리 작업을 처리 가능 (`@BeforeEach`, `@AfterEach`)
- 스프링 테스트 컨텍스트 프레임워크 이용 → 테스트 성능 향상 가능
- 동일한 설정 파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유
- `@Autowired`를 사용하면 컨텍스트의 빈을 테스트 오브젝트에 DI 가능
- 학습 테스트는 기술의 사용 방법을 익히고 이해를 도움
- 오류가 발견되는 경우 버그 테스트를 만들어두면 유용
