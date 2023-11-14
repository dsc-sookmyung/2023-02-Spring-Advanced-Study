# 2. 테스트 

1. UserDaoTest

1) 테스트의 유용성

테스트란 결국 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해서, 만든 코드를 확신할 수 있게 해주는 작업이다. 또한 테스트의 결과가 원하는 대로 나오지 않은 경우에는 코드나 설계에 결함이 있을 수 있음을 알 수 있다. 이를 통해 코드의 결함을 제거해가는 작업, 일명 디버깅을 거치게 되고 결국 최종적으로 테스트가 성공하면 모든 결함이 제거됐다는 확신을 얻을 수 있다.

2) UserDaoTest의 특징

-main() 메소드로 작성된 테스트

```java
public class UserDaoTest {
	public static void main(String[] args) throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicatonContext.xml");

		userDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("user");
		user.setName("한지수")
		user.setPassword("isuHan")

		dao.add(user);

		Sysetem.out.println(user.getId() + "등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());

		System.out.println(user.getId() + "조회 성공")
}
	
}
```

이 테스트 방법에서 가장 돋보이는 건, main() 메소드를 이요해 쉽게 테스트 수행을 가능하게 했다는 점과 테스트할 대상인 UserDao를 직접 호출해서 사용한다는 점이다.

- 웹을 통한 DAO 테스트 방법의 문제점
    
    웹 화면을 통해 값을 입력하고, 기능을 수행하고 결과를 확인하는 방법은 가장 흔히 쓰이는 방법이다.
    
    하지만 하나의 테스트를 수행하는 데 참여하는 클래스와 코드가 너무 많다.
    
- 작은 단위의 테스트
    
    한꺼번에 너무 많은 것을 몰아서 테스트하면 테스트 수행 과정도 복잡해지고 오류가 발생했을 때 정확한 원인을 찾기가 힘들어진다. 
    
    따라서 테스트는 가능하면 작은 단위로 쪼개서 집중해서 할 수 있어야 한다. 테스트의 관심이 다르다면 테스트할 대상을 분리하고 집중해서 접근해야 한다.
    
    UserDaoTest는 한 가지 관심에 지중할 수 있게 작은 단위로 만들어진 테스트다
    
    이렇게 작은 단위의 코드에 대해 테스트를 수행한 것을 단위테스트(unit test)라고 한다.
    
- 자동 수행 테스트코드
    
    UserDaoTest는 자바 클래스의 main() 메소드를 실행하는 가장 간단한 방법으로 테스트의 전 과정이 자동으로 진행된다.
    
    자동으로 수행되는 테스트의 장점은 자주 반복할 수 있다는 것이다. 번거로운 작업이 없고 테스트를 빠르게 실행할 수 있기 때문에 언제든 코드를 수정하고 나서 테스트를 할 수 있다.
    
- 지속적인 개선과 점진적인 개발을 위한 테스트
    
    테스트를 이용하면 새로운 기능도 기대한 대로 동작하는지 확인할 수 있을 뿐 아니라, 기존에 만들어뒀던 기능들이 새로운 기능을 추가하느라 수정한 코드에 영향을 받지 않고 여전히 잘 동작하는지를 확인할 수도 있다.
    

3) UserDaoTest의 문제점

- 수동 확인의 번거로움
    
    UserDaoTest는 테스트를 수행하는 과정과 입력 데이터의 준비를 모두 자동으로 진행하도록 만들어졌지만 여전히 사람이 눈으로 확인하는 과정이 필요하다. 
    
    .add() 에서 user 정보를 DB에 등록하고, 이를 다시 get() 을 이용해서 가져왔을 때 입력한 값과 가져온 값이 일치하는지를 테스트코드는 확인해주지 않는다. 
    
    단지 콘솔에 값만 출력해줄 뿐, 콘솔에 나온 값을 보고 등록과 조회가 성공적으로 되고 있는 지를 확인하는 건 사람의 책임이다. 죽 완전히 자동으로 테스트되는 방법이라고 말할 수 없다
    
- 실행 작업의 번거로움
    
    아무리 간단히 실행 가능한 main() 메소드라고 하더라고 매번 실행하는 것서은 번거로우므로, 좀 더 편리하고 체계적으로 테스트를 실행하고 결과를 확인하는 방법이 필요하다.
    

1. UserDaoTest 개선

1) 테스트 검증의 자동화

.add()에 전달한 User 오브젝트에 담긴 사용자 정보와 get() 을 통해 다시 DB에서 가져온 uSER 오브젝트의 정보가 서로 일지하는지 검증해야 한다. 아래의 코드를 수정한다.

수정 전 테스트 코드

```java
System.out.println(user2.getName());
System.out.println(user2.getPassword());
System.out.println(user2.getId() + " 조회 성공");
```

수정 후 테스트 코드

```java
if (!user.getName().equals(user2.getName())) {
	System.out.println("테스트 실패 (name)")
}
else if(!user.getPassword().equals(user2.getPassword())) {
	System.out.println("테스트 실패 (password)")
}
else {
	System.out.println("조회 성공")
}

```

이렇게 해서 테스트의 수행과 테스트 값 적용, 그리고 결과를 검증하는 것까지 모두 자동화했다.

 

2) 테스트의 효율적인 수행과 결과 관리

main() 메소드를 이용한 테스트 작성 방법만으로는 애플리케이션 규모가 커지고 테스트 개수가 많아지면 테스트를 수행하는 일이 점점 부담이 될 것이다. Junit을 사용하면 자바로 단위 테스트를 만들 때 유용하게 사용할 수 있다.

- JUnit 테스트로 전환
    
    프레임워크의 기본 동작원리가 바로 제어의 역전(loC)이기 때문에, main()메소드도 필요 없고 오브젝트를 만들어서 실행시키는 코드를 만들 필요도 없다
    
- 테스트 메소드 전환
    
    테스트가 main() 메소드로 만들어졌다는 것은 제어권을 직접 갖는다는 의미이기 때문에, 이를 일반 메소드로 옮겨야 한다. 메소드를 public으로 선언하고, 메소드에 @Test 애노테이션을 붙인다.
    
    JUnit 프레임워크에서 동작할 수 있는 테스트 메소드로 전환
    
    ```java
    import org.junit.Test
    ...
    public class UserDaoTest {
    	@Test
    	public void addAndGet() throws SQLException() {
    		ApplicastionContext context = new ClassPathApplicationContext("applicationContext.xml")
    		UserDao dao = context.getBean("userDAO", UserDao.class);
    		...
    }
    	
    }
    ```
    
- 검증 코드 전환
    
    테스트의 결과를 검증하는 if/else 문장을 JUnit이 제공하는 assertThast()을 이용해 전환한다.
    
    이 메소드는 첫 번째 파라미터 값을 뒤에 나오는 매처(matcher)라고 불리는 조건으로 비교해서 일치하면 다음으로 넘어가고, 아니면 테스트가 실패하도록 만들어준다. is() 는 매처의 일좀으로 equasl()로 비교해주는 기능을 가졌다.
    
    JUnit을 적용한 UserDaoTest
    
    ```java
    import static org.hamcrest.CoreMatchers.is;
    import static org.junit.AssertThat;
    ...
    public class UserDaoTest {
    	@Test
    	public void addAndGet() throws SQLException() {
    		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    
    		UserDao dao = context.getBean("userDao", UserDao.class);
    		User user = new User();
    		user.setId("user");
    		user.setName("한지수");
    		user.setPassword("isuHan");
    
    		dao.add(user);
    
    		User user2 = dao.get(user.getId());
    
    		assertThat(user2.getName(), is(user.getName()));
    		assertThat(user2.getPassword(), is(user.getPassword()));
    	}
    }
    ```
    
- JUnit 테스트 실행
    
    스프링 컨테이너와 마찬가지로 JUnit 프레임워크도 자바 코드로 만들어진 프로그램이므로 어디선가 한 번은 JUnit 프레임워크를 시작시켜 줘야 한다.
    
    main() 메소드를 하나 추가하고 그 안에 JUnitCore 클래스의 main메소드를 호출해주는 간단한 코드를 넣는다. 메소드 파라미터에는 @Test 테스트 메소드르 가진 클래스의 이름을 넣어준다.
    
    ```java
    import org.junit.runner.JunitCore;
    ...
    public static void main(String[] args) {
    	JUnitCore.main("springbook.user.dao.UserDaoTest");
    } 
    ```
    
    이 클래스를 실행하면 테스트를 실행해는데 걸린 시간, 테스트 결과, 몇 개의 테스트 메소드가 실행됐는지를 알려준다.
    
1.  개발자를 위한 테스팅 프레임워크 JUnit
    
    1) JUit 테스트 실행 방법
    
    - IDE
        
        이클립스 같은 IDE의 지원을 받으면 JUnit 테스트의 실행과 그 결과를 확인하는 방법이 매우 간편하다.
        
    - 빌드 툴
        
        프로젝트의 빌드를 위해 ANT나 Maven 같은 빌드 툴과 스크립트를 사용하고 있다면, 빌드 툴에서 제공하는 JUnit 플러그인이나 테스크를 이용해 JUnit 테스트를 실행할 수 있다.
        
    
    2) 테스트 결과의 일관성
    
    반복적으로 테스트를 했을 때 테스트가 실패하기도 하고 성공하기도 한다면 좋은 테스트라고 할 수 없다. 코드에 변경사항이 없다면 테스트는 항상 동일항 결과를 내야 한다.
    
    UserDaoTest의 문제는 이전 테스트 때문에 DB에 등록된 중복 데이터가 있을 수 있다는 점이다. 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제해서, 테스트를 수행하기 이전 상태로 만든다.
    
    - deleteAll()의 getCount() 추가
        - deleteAll
            
            USER 테이블의 모든 레코드를 삭제해주는 간단한 기능이 있다.
            
        - getCount()
            
            USER 테이블의 레코드 개수를 돌려준다
            
    - deleteAll()과 getCount() 테스트
        
        기존 addAndGet() 테스트에 deleteAll()과 getCount()를 추가하여 , 테스트의 모든 내용이 삭제되었는지 레코드의 개수를 가져와서 검증한다.
        
        ```java
        @Test
        publlic void addAndGet() throws SQLWxception {
        	...
        	dao.deleteAll();
        	assertThat(dao.getCount(), is(0));
        	
        	UserDao dao = context.getBean("userDao", UserDao.class);
        	
        	User user = new User();
        	user.setId("user");
        	user.setName("한지수");
        	user.setPassword("isuHan");
        
        	dao.add(user);
        	assertThat(dao.getCount(), is(1));
        
        	User user2 = dao.get(user.getId());
        
        	assertThat(user2.getName(), is(user.getName()));
        	assertThat(user2.getPassword(), is(user.getPassword()));
        
        }
        ```
        
    - 동일한 결과를 보장하는 테스트
        
        addAndGet() 테스트를 마치기 직전에 deteAll()을 실행할 수도 있지만, addAndGet() 테스트만 DB를 사용할 것이 아니라면 이전에 어떤 작업을 하다 테스트를 실행했는지 알 수 없기 때문에, 테스트를 하기 전에 테스트 실행에 문제가 되지 않는 상태를 만들어 주는 편이 낫다.
        
    
    3)  포괄적인 테스트
    
    - getCount() 테스트
        
        테스트는 한 번에 한 가지 검증 목적에만 충실한 것이 좋다. 그러므로 getCount()를 위한 새로운 테스트 메소드가 필요하다
        
        조건 : @Test가 붙어 있고 public 접근자가 있으며 리턴 값이 void형이고 파리미터가 없다
        
        시나리오 :  USER 테이블의 데이터를 모두 지우고 getCount()로 레코드 개수가 0임을 확인한다. 3개의 사용자 정보를 하나씩 추가하면서 매번 getCount()의 결과가 하나씩 증가하는지 확인한다. 
        
        getCount() 테스트
        
        ```jsx
        @Test
        public void count() throws SQLException {
        	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        
        	UserDao dao = context.getBean("userDao", UserDao.class);
        	User user1 = new User("user1", "한지수", "isuHan");
        	User user2 = new User("user2", "한한지수", "isuHann");
        	User user3 = new User("user3", "한한한지수", "isuHannn");
        
        	dao.deleteAll();
        	assertThat(dao.getCount(), is(0));
        
        	dao.add(user1);
        	assertThat(dao.getCount(), is(1));
        
        	dao.add(user2);
        	assertThat(dao.getCount(), is(2));
        
        	dao.add(user3);
        	assertThat(dao.getCount(), is(3));
        
        }
        ```
        
        주의할 점은 두 개의 테스트가 어떤 순서로 실행될지는 알 수 없다. JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않는다
        
    - addAndGet() 테스트 보완
        
        add()의 기능은 충분히 검토되었으나, get()에 대한 테스트는 부족하므로 이를 보충한다.
        
        시나리오 : User를 하나 더 추가해서 두 개의 User를 add()하고, 각 User의 id를 파라미터로 전달해서 get()을 실행한다.
        
        get() 테스트 기능을 보완한 addAndGet() 테스트
        
        ```jsx
        @Test
        public void addAndGet() throws SQLException {
        	...
        	UserDao dao = context.getBean("userDao", UserDao.class);
        	User user1 = new User("user1", "한지수", "isuHan");
        	User user2 = new User("user2", "한지수수", "isuHann");
        
        	dao.deleteAll();
        	assertThat(dao.getCount, is(0));
        
        	dao.add(user1);
        	dao.add(user2);
        	assertThat(dao.getCount(), is(2));
        
        	User userget1 = dao.get(user1.getId);
        	assertThat(userget1.getName(), is(user1.getName()));
        	assertThat(userget1.getPassword(), is(User1.getPassword));
        
        	User userget2 = dao.get(user1.getId);
        	assertThat(userget2.getName(), is(user2.getName()));
        	assertThat(userget2.getPassword(), is(User2.getPassword));
        }
        ```
        
    - get() 예외조건에 대한 테스트
        
        주어진 id에 해당하는 정보가 없다는 의미를 가진 예외 클래스가 하나 필요하다.
        
        예외 발생 여부는 메소드를 실행해서 리턴값을 비교하는 방법으로 확인할 수 없다.
        
        JUit은 예외조건 테스트를 위한 expected 엘리먼트 방법을 제공한다. 이를 추가하면, 보통의 테스트와는 반대로 정상적으로 테스트 메소드를 마치면 테스트가 실패하고, expected에서 지정한 예외가 던져지면 테스트가 성공한다.
        
        ```jsx
        @Test(expected=EmptyResultDataAccessException.class)
        public void getUserFailure() throws SQLException {
        	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        
        	UserDao dao = context.getBean("userDao", UserDao.class);
        	dao.deleteAll();
        	assertThat(dao.getCount(), is(0));
        
        	dao.get("unknown_id");
        }
        ```
        
    - 테스트를 성공시키기 위한 코드의 수정
        
        위 테스트가 성공하도록 get() 메소드 코드를 수정해야 한다.
        
        주어진 id에 해당하는 데이터가 없으면 EmptyResulstDastaAcessException을 던지는 get() 메소드를 사용한다.
        
        ```jsx
        public User get(String id) throws SQLException {
        	...
        	ResultSet rs = ps.executeQuery();
        
        	User user = null;
        	if(rs.next()) {
        		user = new User();
        		user.setId(rs.getString("name));
        		user.setPassword(rs.getString("password"));
        	}
        	rs.close();
        	ps.close();
        	c.close();
        
        	if(user ==null) throws new EmptyResultDataAccessException(1);
        	return user;
        }
        ```
        
    - 포괄적인 테스트
        
        자신이 만든 코드에서 발생할 수 있는 다양한 상황과 입력 값을 고려하는 포괄적인 테스트를 만들어야 한다. 특히 테스트를 작성할 때 부정적인 케이스를 먼저 만드는 습관을 들이는 게 좋다.
        
    
    4) 테스트가 이끄는 개발
    
    - 기능 설계를 위한 테스트
        
        테스트에는 만들고 싶은 기능에 대한 조건과 행위, 결과에 대한 내용이 잘 표현되어 있다. 이렇게 비교해보면 테스트 코드는 마치 잘 작성된 하나의 기능 정의서처럼 보인다.
        
        - 조건
            
            단계 : 어떤 조건을 가지고
            
            내용 : 가져올 사용자 정보가 존재하지 않는 경우에
            
        - 행위
            
            단계 : 무엇을 할 때
            
            내용 : 존재하지 않는 id로 get()을 실행하면
            
        - 결과
            
            단계 : 어떤 결과가 나온다
            
            내용 : 특별한 예외가 던져진다
            
    - 테스트 주도 개발
        
        만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법이 있다. 이를 테스트 주도 개발(TDD. Test Driven Development)이라고 한다. 또는 테스트를 코드보다 먼저 작성한다고 해서 테스트 우선 개발(Test First Development)이라고도 한다.
        
        TDD는 테스트를 먼저 만들고 그 테스트가 성공하도록 하는 코드만 만드는 식으로 진행하기 때문에 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있다. 또한 테스트를 작성하는 시간과 애플리케이션 코드를 작성하는 시간의 간격이 짧아진다.
        
    
    5) 테스트 코드 개선
    
    - @Before
        
        클래스 내에 존재하는 각각의 @Test 를 실행하기 전에 매번 실행돼야 하는 메소드를 정의한다. 이를 구현하면 다음과 같다.
        
        ```jsx
        import org.junit.Test;
        ...
        
        public class UserDaoTest {
            private UserDao dao;
        
            @Before
            public void setUp() {
                ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
                this.dao = context.getBean("userDao", UserDao.class);
            }
            ...
        
            @Test
            public void addAndGet() throws SQLException {
                ...
            }
        
            @Test
            public void count() throws SQLException {
                ...
            }
        
            @Test(exptected=EmptyResultDataAccessException.class)
            public void getUserFailure() throws SQLException {
                ...
            }
        }
        ```
        
        JUnit은 @Test가 붙은 메소드를 실행하기 전과 후에 각각 @Befor와 @After가 붙은 메소드를 자동으로 실행한다. 대신 이 메소드를 테스트 메소드에서 직접 호출하지 않기 때문에 서로 주고받을 정보나 오브젝트가 있다면 인스턴수 변수를 이용해야 한다.
        
        또한 각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만든다.
        
        테스트 메소드의 일부에서만 공통적으로 사용되는 코드가 있다면, @Before 보다는 일반적인 메소드 추출 방법을 써서 메소드를 분리하고 테스트 메소드에서 직접 호출해 사용하도록 만든다.
        
    - 픽스처
        
        테스트를 수행하는 데 필요한 정보나 오브젝트UserDaoTest에서는 dao가, add() 메소드에 전달하는 User 오브젝트가 대표적인 픽스처이다.이 러한 오브젝트들은 @Before에서 생성하도록 묶어놓는 것이 좋다.
        
    
    1. 스프링 테스트 적용
        
        @Before 메소드가 테스트 메소드 개수만큼 반복되기 때문에 애플리케이션 컨텍스트도 세 번 만들어진다. 지금은 설정도 간단하고 빈도 몇 개 없어서 별문제 아닌 듯하지만, 빈이 많아지고 복잡해지면 애플리케이션 컨텍스트 생성에 적지 않은 시간이 걸릴 수 있다.
        
        JUnit은 테스트 클래스 전체에 걸쳐 딱 한 번만 실행되는 @BeforeClass 스태틱 메소드를 지원한다. 이 메소드에서 애플리케이션 컨텍스트를 만들어 스태틱 변수에 저장해두고 테스트 메소드에서 사용할 수 있다. 하지만 이보다는 스프링이 직접 제공하는 애플리케이션 컨텍스트 테스트 지원 기능을 사용하는 것이 편하다.
        
    
    1) 테스트를 위한 애플리케이션 컨텍스트 관리
    
    - 스프링 테스트 컨텍스트 프레임워크 적용
        
        `ApplicationContext context = new GenericXmlApplicationContext(“applicationContext.xml”);`
        
        다음의 코드를 제거하고 작성한다,
        
        ```java
        RunWith(SpringJUnit4ClassRunner.class)
        @ContextConfiguration(location=”/applicationContext.xml”)
        public class UserDaoTest{
        	@Autowired
        	private ApplicationContext context;
        	…
        
        	@Before
        	public void setUp(){
        		this.dao = this.context.getBean(“userDao”,UserDao.class);
        		…
        	}
        ```
        
        @RunWith ****는 JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용하는 애노테이션이다.
        
        SpringJUnit4ClassRunner 라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해주면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해준다.
        
    - 테스트 메소드의 컨텍스트 공유
        
        ```java
        @Before
        public void setUp() {
        	System.out.println(this.context);
        	System.out.println(this);
        
        }
        ```
        
        컨텍스트는 세 번 모두 동일하다.
        
        첫 번째 테스트가 실행될 때 최초로 애플리케이션 컨텍스트가 처음 만들어지면서 가장 오랜 시간이 소모되고, 그 다음부터는 이미 만들어진 애플리케이션 컨텍스트를 재사용할 수 있기 때문에 테스트 실행 시간이 매우 짧아진다.
        
    - 테스트 클래스의 컨택스트 공유
        
        여러 개의 테스트 클래스가 있는데 모두 같은 설정파일을 가진 애플리케이션 컨텍스트를 사용한다면, 스프링은 테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유하게 해준다.
        
        → 수백개의 테스트 클래스가 있더라도 모두 같은 설정파일을 사용한다면 테스트 전체에 걸쳐 단 한개의 애플리케이션 컨텍스트만 만들어져 사용된다.
        
        @Autowired 가 붙은 인스턴스 변수가 있으면, 테스트 컨텍스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 빈을 찾는다. 타입이 일치하는 빈이 있으면 인스턴스 변수에 주입한다. 또 별도의 DI 설정 없이 필드의 타입정보를 이용해 빈을 자동으로 가져올 수 있는데, 이런 방법을 타입에 의한 자동와이어링이라고 한다.
        
    
    2) DI와 테스트
    
    UserDao와 DB 커넥션 생성 클래스 사이에는 DataSource라는 인터페이스를 뒀다.
    
    하지만 어떤 경우 절대로 DataSource의 구현 클래스를 바꾸지 않고, 시스템 운영 중에 항상 SimpleDiverDataSource를 통해서만 DB 커넥션을 가져올 것이라고 주장한다면,  *굳이* DataSource 인터페이스를 사용하여 DI를 주입해야 할까?
    
    → 그래도 인터페이스를 두고 DI를 적용해야 한다.
    
    첫째는 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없기 때문이다. 
    
    둘째는 클래스의 구현방식은 바뀌지 않는다고 하더라도 인터페이스를 두고 DI를 적용하게 해두면 다른 차원의 서비스 기능을 도입할 수 있기 때문이다.
    
    셋째는 테스트 때문이다.단지 효율적인 테스트를 손쉽게 만들기 위해서라도 DI를 적용해야한다.
    
- 테스트 코드에 의한 DI
    
    테스트에 운영용DB를 사용하면 절대 안되기때문에, 테스트용 디비를 써야한다.
    
    그렇다고 ApplicationContext.xml설정을 개발자가 테스트할 때는 테스트용 DB를 이용하도록 DataSource를 수정했다가, 서버에 배치할 때는 다시 운영용 DB를 사용하는 DataSource 바꿔주는 방법도 있겠지만, 번거롭기도 하고 위험할 수도있다.
    
    @DirtiesContext 애노테이션은 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려준다.
    
    ```java
    @DirtiesContext
    public class UserDaoTest{
    	@Autowired
    	UserDao dao;
    
    	@Before
    	public void setUp(){
    	…
    	DataSource datasource = new SingleConnectionDataSource(
    	“jdbc:mysql://localhost/testdb”, “spring”, “book”, true);
    	dao.setDataSource(dataSource); // 코드에 의한 수동DI
    	}
    ```
    

     이렇게 준비된 DataSource 오브젝트를 생성하고, 애플리케이션 컨텍스트에서 가져온

dao 오브젝트를 이용해 DI 해줄 수 있다.

- 테스트를 위한 별도의 DI 설정
    
    테스트 코드에서 빈 오브젝트에 수동으로 DI하는 방법은 장점보다 단점이 많다.
    
    1. 코드가 많아져 번거롭다.
    2. 애플리케이션 컨텍스트도 매번 새로 만들어야 하는 부담
    
    그래서 아예 테스트에서 사용될 DataSource 클래스가 빈으로 정의된 테스트 전용 설정파일을 따로 만들어두는 방법을 이용해도 된다. 두 가지 종류의 설정파일을 만들어서 하나는 운영 서버에서, 다른 하나는 테스트에 사용하면 된다.
    
    ```java
    <bean id=”dataSource”
    	class = “org.springframework.jdbc.datasource.SimpleDriverDataSource”>
    	<property name=”driverClass” value=”com.mysql.jdbc.Driver”/>
    	<property name=”url” value=”jdbc:mysql://localhost/testdb”/>
    	<property name=”username” value=”spring”/>
    	<property name=”password” value=”book”/>
    </bean>
    ```
    
    그리고 UserDaoTest 의 @ContextConfiguration 애노테이션에 있는 location 의 값을 바꿔준다.
    
    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(location=”/test-applicationContext.xml”)
    public class UserDaoTest {
    ```
    
- 컨테이너 없는 DI테스트
    
    마지막으로 스프링 컨테이너를 사용하지 않고 테스트를 만드는 방법이다.
    
    ```java
    public class UserDaoTest{
    	UserDao dao; 
    	…
    	
    	@Before 
    	public void setUp(){
    	…
    	dao = new UserDao();
    	DataSource dataSource = new SingleConnectionDataSource(
    	“jdbc:mysql://localhost/testdb”, “spring”, “book”, true);
    	dao.setDataSource(dataSource);
    	}
    ```
    
    @Autowried도 없고 오브젝트 생성, 관계설정 등을 직접 해준다.
    
    테스트를 위한 DataSource를 직접 만드는 번거로움이 있지만 애플리케이션 컨텍스트를 아예 사용하지 않으니 코드는 간단해진다. 하지만 JUnit은 매번 새로운 테스트 오브젝트를 만들기 때문에 매번 새로운 UserDao오브젝트가 만들어진다는 단점도 있다.
    
- DI를 이용한 테스트 방법 선택
    
    세 가지 방법은 모두 상황에 따라 유용하게 쓸 수 있다. 하지만 스프링 컨테이너 없이 사용하는 방법을 가장 우선적으로 고려해야 한다.
    
    -여러 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야 할 경우
    
    → 스프링의 설정을 이용한 DI방식의 테스트를 이용하면 편하다. 테스트에서 애플리케이션 컨텍스트를 사용하는 경우에는 테스트 전용 설정파일을 따로 만들어 각 다른 설정파일을 만들어 사용하는 경우가 일반적이다.
    
    -테스트 설정을 따로 만들었다고 하더라도 예외적인 의존관계를 강제로 구성해서 테스트해야 할 경우
    
    → 이때는 컨텍스트에서 DI 받은 오브젝트에 다시 테스트 코드로 수동 DI해서 테스트하는 방법을 사용하면 된다.
    

5. 학습 테스트로 배우는 스프링

보통은 자신이 작성한 프로그램에 대한 테스트 코드를 작성하지만 자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트를 작성해야 한다.

이런 테스트를 학습테스트라고 한다.

학습 테스트의 목적은 자신이 테스트를 만들려고 하는 기술이나 기능에 대해 얼마나 제대로 이해하고 있는지,그 사용 방법을 바로 알고 있는지를 검증하려는 게 목적이다.

또, 테스트 코드를 작성해보면서 빠르고 정확하게 사용법을 익히는 것도 학습 테스트를 작성하는 하나의 목적이다.

- 학습 테스트의 장점
    - 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.
    - 학습 테스트 코드를 개발 중에 참고할 수 있다.
    - 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.
    - 테스트 작성에 대한 좋은 훈련이 된다.
    - 새로운 기술을 공부하는 과정이 즐거워진다.
- 버그테스트
    - 코드에 오류가 있을 때 그 오류를 가장 잘 드러낼 수 있는 테스트
    - 버그 테스트는 일단 실패하도록 만들어야 한다.
    - 그러고 나서 버그 테스트가 성공할 수 있도록 애플리케이션 코드를 수정한다.
    - 장점은 다음과 같다.
    - 테스트의 완성도를 높여주고, 버그의 내용을 명확하게 분석하게 해주며, 기술적인 문제를 해결하는 데 도움이 된다.