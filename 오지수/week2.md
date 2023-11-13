# 2장 테스트

## 🌿 UserDaoTest 다시보기

### 🌱 테스트의 유용성

테스트란 결국 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해서, 만든 코드를 확신할 수 있게 해주는 작업.

→ 디버깅 과정을 거치며 코드와 설계의 결함을 고친다!

이를 통해 테스트가 성공하면 모든 결함이 제거됐다는 확신을 얻을 수 있음.

### 🌱 UserDaoTest의 특징

- main() 메소드로 작성된 UserDaoTest()

```java
public class UserDaoTest {
    public static void main(String[] args) throws SQLException{
        Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);

        User user = new User();
        user.setId("user");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);

        System.out.println(user.getId() + " 등록 성공");

        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());
        System.out.println(user2.getPassword());

        System.out.println(user2.getId() + " 조회 성공");
    }

}

```

#### 웹을 통한 DAO 테스트 방법의 문제점

웹을 통한 DAO 테스트? <br>
→ 웹 화면을 통해 값을 입력하고 기능을 수행하고 결과를 확인하는 방법.

**단점**

- 서비스 클래스 + 컨트롤러 + JSP 뷰 등 모든 레이어의 기능을 다 만들고 나서야 테스트가 가능함.
- 테스트를 하던 중에 에러가 발생하면 어디에서 문제가 발생했는지 일일히 찾아내야 함.
  - 하나의 테스트에 참여하는 클래스와 코드가 너무 많기 때문!

테스트하고 싶은건 UserDao지만 다른 계층의 코드, 컴포넌트, 서버 설정 상태까지 모두 테스트에 영향을 줄 수 있음. <br>
→ 테스트가 번거로워짐 + 오류에 대한 빠르고 정확한 대처가 불가능해짐.

#### 작은 단위의 테스트

테스트는 "테스트하고자 하는 대상"에만 집중해서 테스트를 진행해야 한다. <br>
→ 작은 단위의 테스트로 쪼개서 집중해야 한다! = 관심사의 분리

> **단위 테스트** <br>
> 작은 단위의 코드에 대해 테스트를 수행한 것.

- 여기서 단위의 기준은 **"충분히 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위"** 이다.
  - 단위는 작을 수록 좋다.
  - 다른 코드들과는 독립적으로 실행되어야 함.

테스트 중에 DB가 사용되어도 단위 테스트라고 말할 수 있을까? <br>
→ UserDaoTest를 수행할 때 매번 USER 테이블의 내용을 비우고 테스트를 진행함.

- 사용할 DB의 상태를 테스트가 관장하고 있으므로 단위테스트라고 말할 수 있음!
  - 대신 DB의 상태가 매번 달라지고, 테스트를 위해 DB를 특정 상태로 만들어줄 수 없다면 그때는 가치가 사라지는 것.

> **통제할 수 없는 외부의 리소스에 의존하는 테스트는 단위 테스트가 아니라고 보는 것**이다!

또한 이렇게 각 단위별 테스트를 진행하고 난 뒤라면, <br>
이후 전 과정을 하나로 묶어서 진행하는 테스트를 실행할 때도 훨씬 편하게 디버깅을 수행할 수 있음.

<br>

작은 단위로 나눠서 테스트를 진행하는 이유? <br>
: 설계한 코드가 의도대로 동작하는지 빠르게 확인할 수 있기 때문.

- 이때 확인의 대상과 조건이 간단하고, 명확할 수록 좋음. 그러므로 **단위**테스트를 진행하는 것.
- 그리고 이는 더 빠르고 쉬운 원인을 찾는 걸 도움.

<br>

테스트는 자동으로 수행되도록 코드로 만들어지는 것이 중요하며, <br> 별도의 테스트용 클래스를 만들어서 넣는 편이 좋음.

- 자동으로 수행? 자주 반복 가능하다는 것!
- 전체 테스트를 진행하며 코드를 수정하면 기능 문제가 없는지 빠르게 확인할 수도 있음.

#### 지속적인 개선과 점진적인 개발을 위한 테스트

지속적인 수정 및 설계 과정에서의 확신을 가져다 줌.

- 개선 과정의 속도를 향상시킴.

### 🌱 UserDaoTest의 문제점

하지만 아직 만족스럽지 못한 부분이 있음.

#### 수동 확인 작업의 번거로움

여전히 사람의 눈으로 확인해야 하는 작업이 필요함.

값이 일치하는지 테스트 코드는 확인해주지 않고, 직접 콘솔에 출력된 값을 통해 사람이 확인해야 함.

#### 실행 작업의 번거로움

만약 DAO가 수백개가 되고, 그에 대한 main() 메소드도 그만큼 많이 만들어진다면?

전체 main메소드를 일일히 실행하는 수고 + 그 결과를 모두 눈으로 확인해야 하는 작업 모두 힘들어질 것.

## 🌿 UserDaoTest 개선하기

### 🌱 테스트 검증의 자동화

위 테스트 코드의 검증 부분이다. <br> get() 해서 가져온 결과를 사람이 모두 확인하도록 단순히 콘솔에 출력하기만 한다.

```java
System.out.println(user2.getName());
System.out.println(user2.getPassword());

System.out.println(user2.getId() + " 조회 성공");
```

테스트가 실패하는 것에는 두가지 이유가 있다.

1. 테스트 에러: 에러가 발생해서 실패하는 경우
2. 테스트 실패: 결과가 기댓값과 다르게 나오는 경우

에러가 나는 것은 쉽게 파악이 가능하지만, **실패**의 경우 별도의 확인 작업이 필요한 경우인 것.

다음과 같이 기대값까지 결과값과 비교해주는 로직을 추가하여 테스트 코드를 수정하자.

```java
if (!user.getName().equals(user2.getName())) {
    System.out.println("테스트 실패 (name)")
}
else if (!user.getPassword().equals(user2.getPassword())) {
    System.out.println("테스트 실패 (password)")
}
else {
    System.out.println("조회 테스트 성공")
}
```

이제 할 일은 마지막 출력 메세지가 "조회 테스트 성공"으로 나오는 지 확인하는 것 뿐!

### 🌱 테스트의 효율적인 수행과 결과 관리

수백개의 테스트를 실행시키기 위해서는 단순한 main() 메소드로는 한계가 있다.

1. 일정한 패턴을 가진 테스트 생성이 가능하고
2. 많은 테스트를 간단히 실행시킬 수 있으며
3. 테스트 결과를 종합해서 볼 수 있고
4. 테스트가 실패한 곳을 빠르게 확인할 수 있는 테스트 지원 도구가 필요함!

=> **JUnit!!**

#### JUnit 테스트로 전환

JUnit은 프레임워크이다. <br>
프레임워크는 개발자가 만든 클래스에 대한 제어 권한을 넘겨받아서 주도적으로 애플리케이션의 흐름을 제어한다.

- 개발자가 만든 클래스의 오브젝트 생성, 실행하는 일은 프레임워크에 의해 진행됨.
  - 따라서 프레임워크에서는 main()도 필요없는 것!

#### 테스트 메소드 전환

main() 메소드로 만들어졌다는 것? = 제어권을 직접 갖겠다는 의미.

JUnit 프레임워크가 요구하는 조건

1. public 선언
2. 메소드에 @Test 애노테이션 붙이기

따라서 위 조건에 맞게 재구성한 코드는 다음과 같다.

```java
public class UserDaoTest{

    @Test
    public void addAndGet() throws SQLException {
        Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);

        ...
    }
}

```

#### 검증 코드 전환

```java
if (!user.getName().equals(user2.getName())) {...}
```

이 if 문장의 기능을 JUnit 에서 제공해주는 assertThat이라는 메소드를 이용해 다음과 같이 변경할 수 있다.

```java
assertThat(user2.getName(), is(user.getName()));
```

- assertThat() : 첫번째 파라미터의 값을 뒤에 나오는 매처(조건)로 배교해서 일치하는지 확인함.
  - 여기서는 is()라는 매처가 사용되었고, 이는 equals()와 동일한 기능을 가짐.

JUnit에서는 이 assertThat()이 실패하지 않으면 테스트가 성공했다고 인식함.

따라서 수정한 UserDaoTest는 이와 같음.

```java
public class userDaoTest {

    @Test
    public void addAndGet() throws SQLException {
        Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

        UserDao dao = context.getBean("userDao", UserDao.class);

        User user = new User();
        user.setId("user");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()))
        assertThat(user2.getPassword(), is(user.getPassword()))
    }
}
```

#### JUnit 테스트 실행

어딘에든 main() 메소드를 하나 추가하고, 그 안에 JUnitCore 클래스의 main 메소드를 호출해주는 간단한 코드를 넣어주면 됨.

```java
public static void main(String[] args) {
    JUnitCore.main("springbook.user.dao.UserDaoTest");
}
```

반환 출력

- 걸린 시간
- 테스트 결과
- 몇 개의 테스트 메소드가 실행되었는지

만약 assertThat() 검증과정에서 기대한 결과와 다른값이라면? <br>
**AssertionError**를 던진다.

## 🌿 개발자를 위한 테스팅 프레임워크 JUNIT

### 🌱 JUnit 테스트 실행 방법

가장 좋은 JUnit 테스트 실행방법은 자바 IDE에 내장된 JUnit 테스트 지원 도구를 사용하는 것이다.

#### IDE - 이클립스

- 이는 현재 IntelliJ 를 주로 사용하고 있기 때문에 넘어감..

#### 빌드 툴

ANT, Maven과 같은 빌드 툴을 사용하고 있다면, (최근엔 보통 Gradle) <br>
빌드 툴에서 제공되는 JUnit 플러그인이나 태스크를 이용하여 실행 가능함.

### 🌱 테스트 결과의 일관성

아직 아쉬운 점은 남아있다.

UserDaoTest 테스트를 실행하기 전에 **DB의 USER 테이블 데이터를 모두 삭제**해주야 하기 때문!

- 그냥 테스트를 실행했다가는 이전 테스트를 실행했을 때 등록됐던 사용자 정보와 기본키가 중복되어 에러가 발생함.

테스트가 외부 상태에 따라 성공하기도 하고 실패하기도 한다는 것! <br>
이 테스트를 좋은 테스트라고 할 수 없음.

가장 좋은 해결책 <br>
: 테스트가 등록한 사용자 정보를 삭제하여 **테스트를 수행하기 이전 상태**로 만들어주는 것.

#### deleteAll()의 getCount() 추가

- deleteAll(): 모든 레코드를 삭제하는 기능.

```java
public void deleteAll() throws SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement("delete from users");
  ps.executeUpdate();

  ps.close();
  c.close();
}
```

<br>

- getCount(): User 테이블의 레코드 개수를 돌려줌.

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

#### deleteAll()과 getCount()의 테스트

add()와 get()처럼 독립적으로 자동 실행되는 테스트를 만들기가 애매함.

따라서 기존에 만든 addAndGet() 테스트를 확장하는 방법을 사용하는 편이 더 나을 것!

- deleteAll() 이 실행한 직후에 getCount()을 실행하여 0이 나온다면 deleteAll()의 정상작동을 확인할 수 있음.
- 앞에서 이미 검증한 add() 직후에 getCount() 실행하면 1이 나오는지 확인하여 getCount()의 기능을 확인할 수 있음.

```java
public class userDaoTest {

    @Test
    public void addAndGet() throws SQLException {
        ...

        dao.deleteAll()
        assertThat(dao.getCount(), is(0))

        User user = new User();
        user.setId("gyumee");
        user.setName("박성철");
        user.setPassword("springno1");

        dao.add(user);
        assertThat(dao.getCount(), is(1));

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()))
        assertThat(user2.getPassword(), is(user.getPassword()))
  }
}
```

#### 동일한 결과를 보장하는 테스트

위 테스트를 반복해서 여러 번 실행해도 계속 성공함.

물론 다른 방법도 존재함.

- addAndGet() 테스트를 마치기 직전에 **테스트가 변경하거나 추가한 데이터를 모두 원래 상태로** 만들어 주는 것.
  - 특히나 addAndGet() 메서드만 User 테이블을 사용하는 게 아니라면, 이런 방법이 더 나을 것.

### 🌱 포괄적인 테스트

테스트를 안 만드는 것도 위험하지만, 성의 없이 테스트를 만들어서 문제가 있는 코드인데도 테스트가 성공하게 만드는건 더 위험하다.

#### getCount() 테스트

조금 더 꼼꼼한 getCount() 테스트를 작성해보자.

테스트 시나리오

1. USER 테이블의 데이터를 모두 지우기
2. getCount() 로 레코드 개수가 0임을 확인
3. 3개의 사용자 정보를 하나씩 추가하며 매번 getCount() 결과가 하나씩 증가하는지 확인하기

```java
@Test
public void count() throws SQLException {
    Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);

    // User 생성자 추가함
    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");
    User user3 = new User("bumjin", "박범진", "springno3");

    dao.deleteAll()
    assertThat(dao.getCount(), is(0))

    dao.add(user1);
    assertThat(dao.getCount(), is(1))

    dao.add(user2);
    assertThat(dao.getCount(), is(2))

    dao.add(user3);
    assertThat(dao.getCount(), is(3))
}
```

**테스트가 어떤 순서대로 실행될지는 전혀 알 수 없다는 점**을 주의해야 한다!

- 테스트의 결과가 테스트 실행 순서에 영향을 받는다면 테스트를 잘못 만든 것
- 모든 테스트는 **실행 순서에 상관없이 독립적으로 항상 동일한 결과**를 낼 수 있도록 해야 한다.

#### addAndGet() 테스트 보완

get() 메소드에 대한 테스트 기능을 더 보완하자

1. User를 하나 더 추가해서 두 개의 User를 add()하고,
2. 각 User의 id를 파라미터로 전달하여 get()을 실행시키도록 함.

```java
@Test
public void addAndGet() throws SQLException {
    Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

    UserDao dao = context.getBean("userDao", UserDao.class);

    // User 생성자 추가함
    User user1 = new User("gyumee", "박성철", "springno1");
    User user2 = new User("leegw700", "이길원", "springno2");

    dao.deleteAll();
    assertThat(dao.getCount(), is(0));

    dao.add(user1);
    dao.add(user2);
    assertThat(dao.getCount(), is(2));

    User userget1 = dao.get(user1.getId());
    assertThat(userget1.getName(), is(user1.getName()));
    assertThat(userget1.getPassword(), is(user1.getPassword()));

    User userget2 = dao.get(user2.getId());
    assertThat(userget2.getName(), is(user2.getName()));
    assertThat(userget2.getPassword(), is(user2.getPassword()));
}
```

#### get() 예외 조건에 대한 테스트

get() 메소드에 전달된 id 값에 해당하는 사용자 정보가 없다면? <br>
id에 해당하는 정보를 찾을 수 없다고 예외를 던진다!

- 여기서는 스프링이 미리 정의해놓은 데이터 액세스 예외 클래스를 사용한다.
  - `EmptyResultDataAccessException` 예외 사용

이 때는 예외가 '발생'되어야 테스트가 성공인데, 어떻게 작성해야 테스트를 성공시킬 수 있을까?

- `@Test(expected=EmptyResultDataAccessException.class)`를 넣어서 기대하면 예외 클래스를 지정한다.
  > @Test 애노테이션에 expected를 추가해놓으면 보통의 테스트와는 반대로, <br> 정상적으로 테스트를 마치면 테스트가 실패하고, <br> expected에서 지정한 예외가 던져지면 테스트가 성공함.

```java
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException {
  Application context =
                new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);
  dao.deleteAll();

  assetThat(dao.getCount(), is(0));

  dao.get("unknwon_id") // 예외를 발생시키는 아이디
}

```

#### 테스트를 성공시키기 위한 코드의 수정

다시 이 테스트가 성공하도록 get() 메소드 코드를 수정해보자.

```java
public User get(){
  ResultSet rs = ps.executeQuery();

  User user = null;
  if (rs.next()) {
    user = new User();
    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));
  }
  rs.close();
  ps.close();
  c.clse();

  if (user == null) throw new EmptyResultDataAccessException(1);

  return user;
}

```

#### 포괄적인 테스트

이렇게 간단한 DAO라도 포괄적인 테스트를 만들어두는 편이 훨씬 안전하고 유용하다.

개발자가 테스트를 직접 만들 때 자주 하는 실수? **성공하는 테스트만 골라서 만드는 것.**

다양한 경우에 대한 전문적인 테스트가 성공될 필요가 있음.

항상 네거티브 테스트를 먼저 만들라. 부정적인 케이스를 먼저 만드는 습관을 들이는 게 좋음.

- ex) 존재하는 것보다, 존재하지 않는 id가 주어졌을 때 어떻게 반응하는지를 먼저 결정하는 게 더 꼼꼼한 개발을 가능하게 한다!

### 🌱 테스트가 이끄는 개발

테스트할 코드를 먼저 만드는 것이 아닌, **테스트를 먼저 만들고 개발을 진행**한다!

#### 기능설계를 위한 테스트

먼저 만들어진 코드를 보고 어떻게 테스트할지 생각하면서 테스트 코드를 개발하는 것이 아니다. <br>
→ **추가하고 싶은 기능을 테스트 코드로 표현**하고자 해야한다!

- 어떤 조건을 가지고 (조건),
- 무엇을 할 때 (행위),
- 어떤 결과가 나온다 (결과)

→ 마치 잘 작성된 기능 정의서처럼 테스트 코드를 작성해야 한다.

#### 테스트 주도 개발

TDD: Test Driven Development = 테스트 주도 개발 : <br>
만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법

"실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다."

- 아예 테스트를 먼저 만들고, 그 테스트를 성공시키기 위한 코드만 작성하게 하도록 하기 때문에 테스트를 빼먹지 않고 꼼꼼하게 만들 수 있도록 도움.
  - 코드 작성 시간의 간격이 짧아짐.

테스트를 작성하고 이를 성공시키는 코드를 만드는 작업의 주기를 가능한 짧게 가져가도록 권장함.

- 단위테스트 작성을 이끔

장점 중 하나는 코드를 만들어 테스트를 실행하는 간격이 매우 짧다는 점

- 오류는 빨리 발견할수록 좋다. : 쉬운 대응 가능해짐.

### 🌱 테스트 코드 개선

테스트 코드도 리팩토링이 필요하다! <br>
→ 내부구조와 설계를 개선하여 좀 더 깔끔하고 이해하기 쉬우며 변경이 용이한 코드로 만들어야 한다.

#### @Before

테스트를 실행할 때마다 반복되는 준비 작업을 별도의 메소드에 넣어보자.

```java
public class UserDaoTest {
  private UserDao dao;

  @Before // @Test가 실행되기 전에 먼저 실행되어야 하는 메소드를 정의한다.
  public void setUp() {
    ApplicationContext context =
        new GenericXmlApplicationContext("applicationContext.xml");
    this.dao = context.getBean("userDao", UserDao.class);
  }
}

```

테스트 클래스에서 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

1. 테스트 클래스에서 @Test가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다.
2. **테스트 클래스의 오브젝트를 하나 만든다.**

   - 한번 만들어진 테스트 클래스의 오브젝트는 하나의 테스트 메소드를 사용하고 나면 버려짐.
   - 각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 확실히 보장해주기 위해 매번 새로운 오브젝트를 만들게 한 것!

3. `@Before`가 붙은 메소드가 있으면 실행한다.
   - 모든 테스트에서 공통적으로 앞부분에 사용되는 코드가 있다면 작성.
   - But, 일부에서만 공통되는 코드? 그냥 일반적인 메소드 추출 방식을 사용한다.
4. `@Test`가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다.
5. `@After`가 붙은 메소드가 있으면 실행한다.
6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
7. 모든 테스트의 결과를 종합해서 돌려준다.

#### 픽스처

픽스처? 테스트를 수행하는 데 필요한 정보나 오브젝트

여기서는 User오브젝트들을 @Before에서 생성하도록 만드는 게 나음.

```java
public class UserDaoTest {
  private UserDao dao;
  private User user1;
  private User user2;
  private User user3;

  @Before
  public void setUp() {
    ApplicationContext context =
        new GenericXmlApplicationContext("applicationContext.xml");
    this.dao = context.getBean("userDao", UserDao.class);

    this.user1 = new User("gyumee", "박성철", "springno1");
    this.user2 = new User("leegw700", "이길원", "springno2");
    this.user3 = new User("bumjin", "박범진", "springno3");
  }
}

```

## 🌿 스프링 테스트 적용

가능한 한 독립적으로 매번 새로운 오브젝트를 만들어 사용하는 것이 좋지만, <br>
애플리케이션 컨텍스트처럼 생성에 많은 시간과 자원이 소요되는 경우에는 테스트 전체가 공유하는 오브젝트를 만드는 게 좋다!

### 🌱 테스트를 위한 애플리케이션 컨텍스트 관리

UserDaoTest에 스프링의 텍스트 컨텍스트 프레임워크를 적용해보자.

> 참고로 스프링부트에서 JUnit5를 사용한다면 `@SpringBootTest`를 붙이면 되는 것으로 알고 있다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
  @Autowired
  private ApplicationContext context;

  @Before
  public void setUp() {
    this.dao = context.getBean("userDao", UserDao.class);

    this.user1 = new User("gyumee", "박성철", "springno1");
    this.user2 = new User("leegw700", "이길원", "springno2");
    this.user3 = new User("bumjin", "박범진", "springno3");
  }
}

```

- `@RunWith(SpringJUnit4ClassRunner.class)` : 스프링의 테스트 컨텍스트 프레임워크의 JUnit 확장기능 지정
  - JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 관리하는 작업을 진행해줌.
- `@ContextConfiguration(locations="/applicationContext.xml")` : 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 위치 지정
- `@Autowired private ApplicationContext context;` : 테스트 오브젝트가 만들어지고 나면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입됨.

#### 테스트 메소드의 컨텍스트 공유

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
  @Autowired
  private ApplicationContext context;
}

```

위 코드에서 생성된 context는 하나의 애플리케이션 컨텍스트에서 만들어져 모든 테스트 메소드에서 사용되고 있음.

어떻게 구현?

- 테스트가 실행되기 전에 딱 한 번만 Application context를 만들어두고, 테스트 오브젝트가 만들어질 때마다 특별한 방법을 이용해 애플리케이션 컨텍스트 자신을 테스트 오브젝트의 특정 필드에 주입해주는 것.

#### 테스트 클래스의 컨텍스트 공유

여러 개의 테스트 클래스가 있는데 **모두 같은 설정 파일**을 가진 애플리케이션 컨텍스트를 사용한다면, 스프링은 **테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유**하게 해줌.

- 따라서 수백 개의 테스트 클래스를 만들었는데 모두 같은 설정 파일을 사용한다면 테스트 전체에 걸쳐 단 한 개의 애플리케이션 컨텍스트만 만들어져 사용된다.

#### @Autowired

스프링의 DI에 사용되는 특별한 애노테이션

- `@Autowired`가 붙은 인스턴스 변수가 있다면 테스트 컨텍스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 Bean을 찾음.
  - 타입과 일치하는 빈이 있으면 인스턴스 변수에 주입해줌.

즉, 자동 와이어링!

<br>

스프링 애플리케이션 컨텍스트는 초기화할 때, **자기 자신도 빈으로 등록**한다.

따라서 애플리케이션 컨텍스트에는 ApplicationContext 타입의 빈이 존재하는 셈. 그래서 DI도 가능함.

굳이 getBean()을 사용하는 것이 아닌 아예 UserDao 빈을 직접 DI받을 수도 있을 것.

```java
publc class UserDaoTest {
  @Autowired
  UserDao dao;

  ...
}
```

`@Autowired`는 변수에 할당 가능한 타입을 가진 빈을 자동으로 찾는다. <br>
단, 같은 타입의 빈이 두 개 이상 있는 경우에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없다.

테스트는 필요하다면 언제든지 애플리케이션 클래스와 밀접한 관계를 맺고 있어도 상관 없다. 그러므로 필요에 따라 타입 구체화도 상관 X

### 🌱 DI와 테스트

만약 DataSource의 구현 클래스가 운영 중에 항상 SimpleDriverDataSource를 통해서만 DB커넥션을 가져오는 경우에도 굳이 DataSource 인터페이스를 사용하여 DI를 통해 주입해주는 방식을 사용해야 하는가?

→ 해당 경우에도 우리는 DI를 적용해야 한다.

1. 소프트웨어 개발에서 절대 바뀌지 않는 것은 없기 때문이다.
   - 당장에는 클래스를 바꾸서 사용할 계획이 전혀 없더라고, 언젠가 필요한 상황이 닥쳤을 때 수정에 들어가는 시간과 비용의 부담을 줄여줄 수 있다.
2. 다른 차원의 서비스 기능을 도입할 수 있다.
   - 새로운 부가기능을 추가하는 게 용이하다.
3. 테스트 때문이다.
   - 효율적인 테스트를 손쉽게 만들기 위해 DI를 적용해야 함.
   - 단위 테스트 작성을 위해 중요.
   - **DI는 테스트가 작은 단위의 대상에 대해 독립적으로 만들어지고 실행되게 하는 데 중요한 역할**을 함.

#### 테스트 코드에 의한 DI

DI는 스프링 컨테이너에서만 할 수 있는 작업이 아님.

UserDao에는 DI컨테이너가 의존관계 주입에 사용되도록 수정자 메소드를 만들어뒀음.<br>
해당 수정자 메소드는 테스트 코드에서도 얼마든지 호출에서 사용 가능 <br>
→ 따라서 테스트 코드 내에서 이를 이용해 직접 DI가 가능하다.

<br>
테스트용 DB에 연결해주는 DataSource를 테스트 내에서 직접 만들 수 있다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
@DirtiesContext
public class UserDaoTest {
  @Autowired
  private ApplicationContext context;

  @Before
  public void setUp() {
    ...
    DataSource dataSource = new SingleConnectionDataSource(
      "jdbc:mysql://localhost/testdb", "spring", "book", true
    );

  }
}
```

테스트가 진행되는 동안에는 UserDao가 테스트용 DataSoucrce를 사용해서 동작하게 됨.

- `@DirtiesContext`: 테스트 메소드에서 애프리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려준다.

<br>

장점

- XML 설정파일을 수정하지 않고도 테스트 코드를 통해 오브젝트 관계를 재구성할 수 있음.

But, 주의해서 사용해야 함.

- applicationContext.xml 파일의 설정정보를 따라 구성한 오브젝트를 가져와 의존관계를 강제로 변경했기 때문.
- 한 번 변경하면 나머지 모든 테스트를 수행하는 동안 변경된 애플리케이션 컨텍스트가 계속 사용될 것. **바람직하지 못함.**

<br>

그래서 UserDaoTest에서 `@DirtiesContext`를 붙여준 것.

- 해당 테스트 클래스에서 애플리케이션 컨텍스트의 상태를 변경한다는 것을 알림.
- 이 애노테이션이 붙으면 애플리케이션 컨텍스트 공유를 허용하지 않음.
  - 매번 새로운 애플리케이션 컨텍스트를 만들어 뒤 테스트에 영향을 주지 않게 함.

#### 테스트를 위한 별도의 DI 설정

아예 **테스트에서 사용될 DataSource클래스가 빈으로 정의된 테스트 전용 설정파일을 따로 만들어두는 방법**을 이용해도 된다.

test-applicationContext.xml이라고 만들어서 두 개의 설정파일을 사용한다.

그리고 UserDaoTest의 `@ContextConfiguration`을 다음과 같이 설정해준다. <br>

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserDaoTest {
  ...
}
```

#### 컨테이너 없는 DI 테스트

아예 스프링 컨테이너를 사용하지 않고 테스트를 만드는 방법도 존재한다.

- 원한다면 스프링 컨테이너를 이용해서 IoC 방식으로 생성되고 DI되도록 하는 대신, **테스트 코드에서 직접 오브젝트를 만들고 DI해서 사용**해도 된다.

```java
// @RunWith가 없음: 스프링 테스트 컨텍스트 프레임워크를 사용하지 않음.
public class UserDaoTest {
  // @Autowired가 없음.
  UserDao dao;

  ...

  @Before
  public void setUp() {
    ...
    dao = new UserDao(); // 직접 오브젝트 생성
    DataSource dataSource = new SingleConnectionDataSource(
      "jdbc:mysql://localhost/testdb", "spring", "book", true
    );
    dao.setDataSource(dataSource); // 직접 DI를 해줌.
  }
}
```

DataSource를 직접 만든다는 번거로움은 있지만, 코드가 단순하고 이해하기 편해짐.

#### DI를 이용한 테스트 방법 선택

그렇다면 DI를 테스트에 이용하는 세 가지 방법 중 어떤 것을 선택해야 될까?

1. 항상 스프링 컨테이너 없이 테스트할 수 있는 방법을 우선적으로 고려하기
2. 스프링의 설정을 이용한 DI 방식의 테스트는 **여러 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트할 때** 사용한다.
3. DI 받은 오브젝트에 다시 테스트 코드로 수동 DI 하는 방식의 테스트는 테스트 설정을 따로 만들었다고 하더라도 예외적인 의존관계를 강제로 구성해서 태스트해야 하는 경우에 사용한다.

## 🌿 학습 테스트로 배우는 스프링

때로는 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대해서도 테스트를 작성해야 할 때가 있다.

이런 게 바로 **학습 테스트(learning test)**!!

학습 테스트의 목적: <br>
자신이 사용할 API나 프레임워크의 기능을 테스트로 보면서 **사용 방법을 익히려는 것**.

### 🌱 학습 테스트의 장점

#### 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.

학습 테스트는 자동화된 테스트 코드로 만들어지기 때문에 다양한 조건을 설정할 수 있다.

#### 학습 테스트 코드를 개발 중에 참고할 수 있다.

학습 테스트는 다양한 기능과 조건에 대한 테스트 코드를 개별적으로 만들고 남겨둘 수 있다.

따라서 해당 테스트 코드를 실제 개발에서 샘플 코드로 참고할 수 있다.

#### 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.

가끔은 새로운 업데이트를 했을 때 API 사용법에 미묘한 변화가 생길 수 있다.

학습 테스트를 이용하면 새로운 버전의 프레임워크나 제품을 학습 테스트에 먼저 적용해 봄으로써 기능 변화 혹은 버그를 확인할 수 있다.

#### 테스트 작성에 대한 좋은 훈련이 된다.

프레임워크의 학습 테스트는 실제로 애플리케이션 코드의 테스트와 비슷하고, 대체로 단순하다. <br>
→ 한결 작성하기 수월하고 부담도 적음.

따라서 테스트 작성의 훈련 기회로 삼기에 좋음! 또한 새로운 테스트 방법을 연구할 때도 도움이 됨.

#### 새로운 기술을 공부하는 과정이 즐거워진다.

테스트 코드를 만들면서 하는 학습은 흥미롭고 재밌음.

<br>

학습 테스트를 만들 때 참고할 수 있는 가장 소스는 **스프링 자신에 대한 테스트 코드**!!

스프링 배포판의 압축을 풀어보면 프레임워크 소스코드와 함께 테스트 코드도 발견할 수 있음.

- 레퍼런스 문서에서 미처 설명되지 않은 중요한 정보를 많이 얻을 수 있음.
- 테스트 작성 방법에 대한 좋은 팁을 얻어갈 수 있음.

### 🌱 학습 테스트 예제

#### JUnit 테스트 오브젝트 테스트

- JUnit은 정말 테스트 메소드를 수행할 때마다 새로운 오브젝트를 생성할까?

```java
public class JUnitTest {
  static Set<JUnitTest> testObjects = new HashSet<JUnitTest>;

  @Test
  public void test1() {
    assertThat(testObjects, not(hasItem(this)))
    testObjects.add(this);
  }

  @Test
  public void test2() {
    assertThat(testObjects, not(hasItem(this)))
    testObjects.add(this);
  }

  @Test
  public void test3() {
    assertThat(testObjects, not(hasItem(this)))
    testObjects.add(this);
  }

}
```

- `not()`은 뒤에 나오는 결과를 부정하는 매처.

#### 스프링 테스트 컨텍스트 테스트

스프링의 테스트용 애플리케이션 컨텍스트는 테스트 개수에 상관없이 한 개만 만들어진다.

해당 내용에 대한 테스트를 작성해보자.

- 새로운 설정 파일`junit.xml`을 작성하여 테스트에 사용함.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class JUnitTest {
  @Autowired
  ApplicationContext context;

  static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
  static ApplicationContext contextObject = null;

  @Test
  public void test1() {
    assertThat(testObjects, not(hasItem(this)));
    testObjects.add(this);

    assertThat(
      contextObject == null ||
      contextObject == this.context, is(true)
    );
    contextObject = this.context;
  }

  @Test
  public void test2() {
    assertThat(testObjects, not(hasItem(this)));
    testObjects.add(this);

    assertTrue(
      contextObject == null ||
      contextObject == this.context);
    contextObject = this.context;
  }

  @Test
  public void test3() {
    assertThat(testObjects, not(hasItem(this)));
    testObjects.add(this);

    assertThat(contextObject,
      either(is(nullValue())).or(is(this.context))
    );
    contextObject = this.context;
  }
}

```

다양한 검증 방법들

1. `assertThat()`을 이용하기

   - 첫 번째 파라미터에 조건문을 넣고,
   - 그 결과를 `is()` 매처를 써서 true와 비교하기

2. 조건문을 받아서 그 결과가 true인지 false인지를 확인하기

3. `assertThat()`와 매처와 조합하기

   - `either()`: 뒤에 이어서 나오는 or()과 함께 두 개의 매처 결과를 OR 조건으로 비교해줌.
     - 두 매처 중에 하나만 true 값으로 나와도 성공.

### 🌱 버그 테스트

버그 테스트는 실패하도록 만들어야 한다.

버그가 원인이 되어 테스트가 실패하도록 코드를 작성해야 함!

#### 테스트의 완성도를 높여준다.

기존 테스트에서는 미처 검증하지 못했던 부분이 있었기에 오류가 발견된 것.

기존의 불충분했던 테스트를 보완해주는 역할을 함.

#### 버그의 내용을 명확하게 분석하게 해준다.

버그를 효과적으로 분석할 수 있으며 또 그 과정에서 해당 버그로 발생하는 다른 오류를 발견할 수도 있음.

#### 기술적인 문제를 해결하는 데 도움이 된다.

기술적으로 다루기 힘든 버그를 발견했을 때 그에 대한 버그 테스트를 만들어보면 도움이 됨.
