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

아직 불편한 점은 남아있다.

### 🌱 포괄적인 테스트

### 🌱 테스트가 이끄는 개발

### 🌱 테스트 코드 개선

## 🌿 스프링 테스트 적용

### 🌱 테스트를 위한 애플리케이션 컨텍스트 관리

### 🌱 DI와 테스트

## 🌿 학습 테스트로 배우는 스프링

### 🌱 학습 테스트의 장점

### 🌱 학습 테스트 예제

### 🌱 버그 테스트
