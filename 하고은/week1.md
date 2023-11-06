## 오브젝트
- 스프링이 가장 관심을 많이 두는 대상
- 오브젝트에 대한 관심은 오브젝트의 설계로 발전함
  - 객체 지향 설계
  - 디자인 패턴: 재활용 가능한 설계 방법
  - 리팩토링: 깔끔한 구조를 위한 지속적인 개선
  - 단위 테스트: 오브젝트의 동작 검증

</br>

# 1️⃣ DAO의 분리

## ◾ 관심사의 분리

- 미래를 위한 설계와 개발
  - 객체 지향 프로그래밍은 변화에 효과적으로 대처할 수 있음
    → 가상의 추상 세계 자체를 효과적으로 구성하고 편리하게 변경, 발전, 확장 가능
  - 변화에 좋은 대책은 변화의 폭을 줄이는 것
    → **분리와 확장**을 고려한 설계
- 분리: 모든 변경과 발전은 한 번에 한 가지 관심사항에 집중해서 일어나지만 작업은 한 곳에 집중되어 일어나지 않음
  → 관심사의 분리 필요: 관심이 같은 것은 모으고 다른 것은 분리하면 같은 관심에 효과적으로 집중이 가능함

</br>

## ◾ 커넥션 만들기의 추출

### 중복 코드의 메소드 추출

```java
public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
}

public void get(String id) throws ClassNotFoundException, SQLException {
		Connection c = getConnection();
		...
}

private Connection getConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");
		return c;
}
```

</br>

**💡 리팩토링**
<aside>

- 기존의 코드를 외부의 동작 방식 변화 없이 내부 구조를 변경해서 재구성하는 작업 또는 기술
- 코드 내부 개선과 코드 이해가 편해지며 효율적인 변화 대응 가능
</aside>

</br>

## ◾ DB 커넥션 만들기의 독립

### 상속을 통한 확장

- 템플릿 메소드 패턴: 슈퍼 클래스에 기본적인 로직 흐름을 만들고 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 후 서브 클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하는 방법
  - 슈퍼 클래스: 변하지 않는 기능
  - 서브 클래스: 자주 변경하고 확장할 기능
  - 추상 메소드 구현, 훅 메소드(슈퍼 클래스에서 디폴트 기능을 정의해두거나 비워두었다가 서브 클래스에서 선택적으로 오버라이드 할 수 있도록 한 메소드) 오버라이드
- 팩토리 메소드 패턴: 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것, 상속을 통한 기능 확장
  - 팩토리 메소드: 서브클래스에서 오브젝트 생성 방법과 클래스 결정 가능하도록 미리 정의해둔 메소드
  - 오브젝트 생성 방법을 슈퍼클래스의 기본 코드에서 독립시킴
- 관심사 분리 UserDao: 어떤 기능을 사용한다는 데에 관심
- NUserDao, DUserDao: 어떤 식으로 Connection 기능을 제공할지

```java
public abstract class UserDao {
		public void add(User user) throws ClassNotFoundException, SQLException {
				Connection c = getConnection();
				...
		}

		public void get(String id) throws ClassNotFoundException, SQLException {
				Connection c = getConnection();
				...
		}

		// 추상 메소드 사용
		public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}

public class NUserDao extends UserDao {
		public Connection getConnection() throws ClassNotFoundException, SQLException {
		}
}

public class DUserDao extends UserDao {
		public Connection getConnection() throws ClassNotFoundException, SQLException {
		}
}
```

</br>

**💡 디자인 패턴**

<aside>
	
- 주로 객체 지향 설계에 관한 것
- 객체 지향 설계 문제를 해결하기 위한 확장성 추구 방법
  1.  클래스 상속
  2.  오브젝트 합성
      **_→ 디자인 패턴이 대부분 비슷함_**
- 패턴의 핵심이 담긴 목적 또는 의도 파악 중요 (패턴을 적용할 상황, 해결할 문제, 솔루션의 구조, 각 요소의 역할 + 핵심의도 기억하기)
</aside>

</br>

# 2️⃣ DAO의 확장

## ◾ 클래스의 분리

- DB 커넥션과 관련된 부분을 별도의 클래스에 담음 (독립적인 클래스)
  **→ _SimpleConnectionMaker_**

```java
// SimpleConnectionMaker라는 특정 클래스에 종속
public class UserDto {
		private SimpleConnectionMaker simpleConnectionMaker;

		// 🚨 다른 방식으로 DB 커넥션을 제공하는 클래스를 사용하기 위해 수정이 필요
		// 🚨 DB 커넥션을 제공하는 클래스가 어떤 것인지 UserDao가 구체적으로 알고 있어야 함
		public UserDao() {
				simpleConnectionMaker = new SimpleConnectionMaker();
		}

		public void add(User user) throws ClassNotFoundException, SQLException {
				// 🚨 다른 DB 커넥션 메소드를 사용하려면 수정 필요
				Connection c = simpleConnectionMaker.makeNewConnection();
				...
		}

		public void get(String id) throws ClassNotFoundException, SQLException {
				// 🚨 다른 DB 커넥션 메소드를 사용하려면 수정 필요
				Connection c = simpleConnectionMaker.makeNewConnection();
				...
		}
}
```

```java
public class SimpleConnectionMaker {
    public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
        return c;
    }
}
```

</br>

## ◾ 인터페이스의 도입

- 추상화: 공통적인 성격을 뽑아내어 따로 분리하는 작업 (인터페이스)
  - 기능(어떤 일을 하겠다) 정의, 구현 방법(어떻게 하겠다) 나타나 있지 않음
    → 구현할 클래스의 몫

```java
public interface ConnectionMaker {
		public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

```java
public class DConnectionMaker implements ConnectionMaker{

    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
        return c;
    }
}
```

```java
public class UserDao {
		// 구체적인 클래스 방법을 몰라도 됨
		private ConnectionMaker connectionMaker;

		public UserDao() {
				// 🚨 구체적인 클래스 호출
				// UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계를 설정해주는 것에 대한 관심
				connectionMaker = new DConnectionMaker();
		}

		public void add(User user) throws ClassNotFoundException, SQLException {
				// 인터페이스에 정의된 메소드를 사용 -> 메소드 이름이 변경될 걱정 X
				Connection c = connectionMaker.makeConnection();
				...
		}

		public void get(String id) throws ClassNotFoundException, SQLException {
				Connection c = connectionMaker.makeConnection();
				...
		}
}
```

</br>

## ◾ 관계설정 책임의 분리

- 서비스: 사용되는 오브젝트
- 클라이언트: 사용하는 오브젝트
- UserDao의 클라이언트에서 UserDao를 사용하기 전에 UserDao가 어떤 ConnectionMaker의 구현 클래스를 사용할지 결정하게 만들기
- UserDao 오브젝트와 특정 클래스로부터 만들어진 ConnectionMaker 오브젝트 사이에 관계를 설정해주는 것 (**_오브젝트와 오브젝트 사이의 관계 설정 ⭕_**, 클래스 사이의 관계 설정 ❌)
  - 클래스 사이에 관계가 만들어진다는 것은 한 클래스가 인터페이스 없이 다른 클래스를 직접 사용한다는 것
- UserDao 오브젝트가 DConnectionManager() 오브젝트를 사용하게 하려면 두 클래스의 오브젝트 사이에 런타임 사용관계 (링크, 의존관계)를 맺어주어야 함
  - 모델링 시에는 없고 오브젝트로 만들어진 후 생성되는 관계임
- 클라이언트는 UserDao의 ConnectionMaker의 구현 클래스를 선택하고 선택한 클래스의 오브젝트를 생성해서 UserDao와 연결 (기존에는 UserDao가 하던 것을 해줌)

```java
public UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
}
```

```java
// UserDao와 ConnectionMaker 구현 클래스와의 런타임 오브젝트 의존 관계를 설정하는 책임 담당
public class UserDaoTest {
		public static void main(String[] args) throws ClassNotFoundException, SQLException {
				// UserDao가 사용할 ConnectionMaker의 구현 클래스 결정
				ConnectionMaker connectionMaker = new DConnectionMaker();

				// 1. UserDao 생성
				// 2. 사용할 ConnectionMaker 타입의 오브젝트 제공 -> 두 오브젝트 사이의 의존관계 설정 효과
				UserDao dao = new UserDao(connectionMaker);

				...
		}
}
```

</br>

## ◾ 원칙과 패턴

### 개방 폐쇄 원칙(OCP)

- 클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 함
  - ex) UserDao는 DB 연결 방법이라는 기능을 확장하는 데는 열려 있지만 자신의 핵심 기능을 구현한 코드는 변화에 영향을 받지 않으므로 변경에는 닫혀 있음

### 높은 응집도와 낮은 결합도

- 높은 응집도: 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있음 (패키지, 컴포넌트, 모듈 등 대상의 크기가 달라져도 동일한 원리로 적용 가능)
- 낮은 결합도: 책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태를 유지

### 전략 패턴

- 자신의 기능 맥락(컨텍스트 ex.UserDao)에서 필요에 따라 변경이 필요한 알고리즘(독립적인 책임으로 분리가 가능한 기능)을 인터페이스를 통해 통째로 외부로 분리하고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 함

</br>

# 3️⃣ 제어의 역전(IoC)

## 오브젝트 팩토리

### 팩토리

- 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 오브젝트 (추상 팩토리 패턴이나 팩토리 메소드 패턴과 다름)
- 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 깔끔하게 분리

```java
// UserDao의 생성 책임을 맡은 팩토리 클래스
public class DaoFactory {
		public UserDao userDao() {
				ConnectionMaker connectionMaker = new DConnectionMaker();
				UserDao userDao = new UserDao(connectionMaker);
				return userDao;
		}
}
```

```java
public class UserDaoTest {
		public static void main(String[] args) throws ClassNotFoundException, SQLException {
				UserDao dao = new DaoFactory().userDao();
		}
}
```

</br>

### 설계도로서의 팩토리

- UserDao, ConnectionMaker: 애플리케이션의 핵심적인 데이터 로직과 기술 로직 담당 (실질적인 로직을 담당하는 컴포넌트)
- DaoFactory: 애플리케이션의 오브젝트들을 구성하고 관계를 정의하는 책임(컴포넌트의 구조와 관계를 정의한 설계도)

</br>

## ◾ 오브젝트 팩토리의 활용

- DaoFactory에 UserDao가 아닌 다른 DAO의 생성 기능을 넣을 경우 ConnectionMaker 구현 클래스의 오브젝트를 생성하는 코드가 반복됨

```java
public class DaoFactory {
    public UserDao userDao() {
        return new UserDao(new DConnectionMaker());
    }
    public AccountDao userDao() {
        return new AccountDao(new DConnectionMaker());
    }
    public MessageDao userDao() {
        return new MessageDao(new DConnectionMaker());
    }
}
```

- 메소드 추출

```java
public class DaoFactory {
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }
    public AccountDao userDao() {
        return new AccountDao(connectionMaker());
    }
    public MessageDao userDao() {
        return new MessageDao(connectionMaker());
    }
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```

</br>

### 제어권의 이전을 통한 제어관계 역전

- 프로그램의 제어 프름 구조가 뒤바뀌는 것
- 기본 흐름: 모든 오브젝트가 능동적으로 자신이 사용할 클래스를 결정하고 언제, 어떻게 오브젝트를 만들지 스스로 관장
- 제어의 역전: 모든 오브젝트는 위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정 & 만들어짐 (프로그램의 시작을 담당하는 main()과 같은 엔트리 포인트 제외)
  - ex) 컨테이너 안에서 동작하는 구조(서블릿, JSP, EJB), 디자인 패턴(템플릿 메소드 패턴), 프레임 워크(프레임워크 위에 개발한 클래스를 등록하고 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용)
  - UserDao(수동적으로 사용)와 DaoFactory

</br>

# 4️⃣ 스프링의 IoC

## ◾ 오브젝트 팩토리를 이용한 스프링 IoC

### 애플리케이션 컨텍스트와 설정 정보

- 빈: 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
  - 오브젝트 단위의 애플리케이션 컴포넌트로 스프링 컨테이너가 생성과 관계 설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트를 가리키는 말
- 빈 팩토리: 빈의 생성과 관계 설정 같은 제어를 담당하는 IoC 오브젝트
  - 보통 빈 팩토리를 확장한 애플리케이션 컨텍스트를 주로 사용
- 애플리케이션 컨텍스트: 애플리케이션 전반에 걸쳐 모든 구성 요소의 제어 작업을 담당하는 IoC 엔진의 의미, **별도의 정보를 참고**해서 빈 생성, 관계 설정 등의 제어 작업 총괄
  - 애플리케이션도 애플리케이션 컨텍스트와 설정 정보를 따라서 만들어지고 구성됨

</br>

### DaoFactor를 사용하는 애플리케이션 컨텍스트

- 스프링 프레임워크의 빈 팩토리(애플리케이션 컨텍스트)가 IoC 방식의 기능을 제공할 때 사용할 설정 정보

```java
// 애플리케이션 컨텍스트(빈 팩토리)가 사용할 설정 정보
@Configuration
public class DaoFactory {

		// @Bean: 오브젝트 생성을 담당하는 IoC용 메소드
    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker);
    }

		@Bean
		public ConnectionMaker connectionMaker() {
				return new DConnectionMaker();
		}
}
```

```java
public class UserDaoTest {

    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        // 1. @Configuration이 붙은 자바 코드를 설정 정보로 사용
				ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
				// getBean(): ApplicationContext가 관리하는 오브젝트를 요청하는 메소드
				// "userDao": ApplicationContext에 등록된 빈의 이름
				UserDao dao = context.getBean("userDao", UserDao.class);

        // method 02. UserDaoTest 와 동일 패키지 위치를 기점을 찾고 싶을 경우.
        ApplicationContext ac = new GenericXmlApplicationContext("/applicationContext.xml");
```

</br>

### 애플리케이션 컨텍스트의 동작 방식

- 애플리케이션 컨텍스트 = IoC 컨테이너 = 스프링 컨테이너 = 빈 팩토리 = 스프링이라고 부르는 개발자도 있음
- 애플리케이션에서 IoC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계 설정을 담당
- 직접 오브젝트를 생성하고 관계를 맺어주는 코드가 없음
- 생성정보와 연관관계 정보를 별도의 설정정보를 통해 얻음
- 외부의 오브젝트 팩토리에 작업을 위임하고 결과를 가져다가 사용하기도 함
- @Configuration이 붙은 DaoFactory는 애플리케이션 컨텍스트가 사용하는 IoC 설정 정보임
- @Bean이 붙은 메소드의 이름을 가져와 빈 목록을 만들어 둠
- 클라이언트가 getBean() 메소드를 호출하면 자신의 빈 목록에서 요청한 이름이 있는지 찾고 있다면 빈 생성 메소드를 호출해서 오브젝트를 생성한 후 클라이언트에게 돌려줌
- 범용적이고 유연한 방법으로 IoC 기능 확장 가능

</br>

**📌 애플리케이션을 사용할 때 장점**

<aside>

- 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없음
- 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해줌
- 빈을 검색하는 다양한 방법을 제공
</aside>

</br>

### 스프링 IoC의 용어 정리

- 빈: 스프링이 IoC 방식으로 관리하는 오브젝트 (관리되는 오브젝트)
  - 스프링을 사용하는 모든 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 아님
- 빈 팩토리: IoC를 담당하는 핵심 컨테이너, 빈 등록, 생성, 조회, 반환, 빈 관리 기능
- 애플리케이션 컨텍스트: 빈 팩토리를 확장한 IoC 컨테이너, 빈 등록, 관리 + 스프링이 제공하는 각종 부가 서비스 추가 제공
  - ApplicationContext: 애플리케이션 컨텍스트가 구현해야 하는 기본 인터페이스를 가리키는 것이기도 함, BeanFactory 상속
- 설정정보/설정 메타정보: 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC 적용하기 위해 사용하는 메타 정보 (구성 정보, 형상 정보)
  - 컨테이너에 어떤 기능을 세팅하거나 조정하는 경우에도 사용 그보다는 IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용
  - 애플리케이션의 형상정보, 애플리케이션의 전체 그림이 그려진 청사진
- 컨테이너/IoC 컨테이너: 애플리케이션 컨텍스트나 빈 팩토리를 IoC 방식으로 빈 관리를 한다는 의미에서 컨테이너 또는 IoC 컨테이너라고 함
  - IoC 컨테이너: 빈 팩토리 관점
  - 컨테이너/스프링 컨테이너: 애플리케이션 컨텍스트 관점
- 스프링 프레임워크: IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말할 때 사용 = 스프링

</br>

# 5️⃣ 싱글톤 레지스트리와 오브젝트 스코프

**📌 오브젝트의 동일성과 동등성**

<aside>

- 동일성 : 두 개의 오브젝트가 완전히 같은 오브젝트 (==)
  - 하나의 오브젝트만 존재, 두 개의 오브젝트 레퍼런스 변수만 있는 상태
- 동등성: 두 개의 오브젝트가 동일한 정보를 담고 있음 (equals())
  - 메모리상에 각기 다른 오브젝트가 존재
- equals() 메소드 재정의하지 않으면 최상위 클래스 (Object 클래스) equals() 메소드가 사용됨 - 동일성 비교 (동일한 오브젝트여야 동등한 오브젝트로 여겨짐)
</aside>

</br>

## ◾ 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트는 IoC 컨테이너이자 싱글톤을 저장하고 관리하는 싱글톤 레지스트리
- 별다른 설정을 하지 않으면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만듦

</br>

### 서버 애플리케이션과 싱글톤

- 싱글톤으로 만드는 이유: 스프링이 적용되는 대상이 자바 엔터프라이즈 기술을 사용하는 서버 환경이기 때문
- 서블릿: 자바 엔터프라이즈 기술의 가장 기본이 되는 서비스 오브젝트
  - 대부분 멀티 스레드 환경에서 싱글톤으로 동작
  - 서블릿 클래스당 하나의 오브젝트만 만들어두고 사용자의 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용 (제한된 수의 오브젝트만 만들어서 사용)

</br>

**📌싱글톤 패턴**
<aside>

- 어떤 클래스를 애플리케이션 내에서 제한된 인스턴스 개수, 이름처럼 주로 하나만 존재하도록 강제하는 패턴
- 만들어진 클래스 오브젝트는 애플리케이션 내에서 전역적으로 접근 가능
- 단일 오브젝트만 존재, 이를 애플리케이션 내의 여러 곳에서 공유하는 경우에 사용
</aside>

</br>

### 싱글톤 패턴의 한계

- 클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private으로 만든다.
- 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의한다.
- 스태틱 팩토리 메소드인 getInstance()를 만들고 이 메소드가 최초로 호출되는 시점에서 한 번만 오브젝트가 만들어지게 한다. 생성된 오브젝트는 스태틱 필드에 저장된다. 또는 스태틱 필드의 초기값으로 오브젝트를 미리 만들어둘 수도 있다.
- 한번 오브젝트(싱글톤)가 만들어지고 난 후에는 getInstance() 메소드를 통해 이미 만들어져 스태틱 필드에 저장해둔 오브젝트를 넘겨준다.

```java
public class UserDao {
    private static UserDao INSTANCE;

    private UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }

    public static synchronized UserDao getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new UerDao(??);
        }
        return INSTANCE;
    }
    ...
}
```

- private 생성자를 갖고 있기 때문에 상속할 수 없다.
- 싱글톤은 테스트하기 힘들다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다.

</br>

### 싱글톤 레지스트리

- 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능 제공
- 스프링 컨테이너는 싱글톤을 생성, 관리, 공급하는 싱글톤 관리 컨테이너이기도 함
- 평범한 자바 클래스를 싱글톤으로 활용 가능
  - IoC 방식의 컨테이너를 사용해서 생성과 관계설정, 사용 등에 대한 제어권을 넘기면 손쉽게 싱글톤 방식으로 만들어져 관리 가능
- public 생성자 가능
  - 싱글톤으로 사용돼야 하는 환경이 아니라면 간단히 오브젝트를 생성해서 사용 가능(ex. 테스트 환경)
- 스프링이 지지하는 객체 지향적인 설계 방식과 원칙, 디자인 패턴 적용에 제약이 없음

</br>

### 싱글톤과 오브젝트의 상태

- 싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에는 무상태(상태 정보를 내부에 갖고 있지 않음) 방식으로 만들어져야 함
- 각 요청에 대한 정보, DB, 서버의 리소스로부터 생성한 정보는 파라미터와 로컬 변수, 리턴 값을 이용 (ex. 메소드 파라미터와 메소드 안의 로컬 변수는 매번 독립적인 공간이 만들어짐)

```java
public class UserDao {
		// 읽기 전용 인스턴스 변수 (다른 싱글톤 빈 저장) -> 문제 없음
		private ConnectionMaker connectionMaker;
		// 🚨 매번 새로운 값으로 바뀌는 정보를 담는 인스턴스 (c, user) -> 심각한 문제 발생
		private Connection c;
		private User user;

		public void get(String id) throws ClassNotFoundException, SQLException {
				Connection c = connectionMaker.makeConnection();
				...
		}
}
```

</br>

### 스프링 빈의 스코프

- 빈의 스코프: 스프링이 관리하는 오브젝트인 빈의 생성, 존재, 적용되는 범위
- 싱글톤 스코프: 기본 스코프, 컨테이너 내에 한 개의 오브젝트가 만들어진 후 스프링 컨테이너가 존재하는 한 계속 유지
- 프로토타입 스코프: 컨테이너에 빈을 요청할 때마다 새로운 오브젝트를 만들어줌
- 요청 스코프: 웹을 통해 HTTP 요청이 생길 때마다 생성
- 세션 스코프: 웹의 세션과 스코프가 유사

</br>

# 6️⃣ 의존관계 주입(DI)

## ◾ 제어의 역전(IoC)과 의존관계 주입

- 의존관계 주입(DI): 스프링 IoC 기능의 대표적인 동작 원리
  - 과거: IoC 컨테이너 → 현재: 의존관계 주입 컨테이너, DI 컨테이너라고 불림
- 의존(종속) 오브젝트 주입이라고 부르기도 하지만 오브젝트는 주입될 수 있는 것이 아님
  - DI는 오브젝트 **레퍼런스**를 외부로부터 제공받고 이를 통해 디른 오브젝트와 **다이나믹**하게 의존관계가 만들어지는 것이 핵심
  - 주입받는 메소드 파라미터가 특정 클래스 타입으로 고정되면 안됨 → 인터페이스 필요

</br>

## ◾ 런타임 의존관계 설정

### 의존 관계

- 방향성이 필요함
- A가 B에 의존하고 B는 A에 의존하지 않음 (= B는 A의 변화에 영향을 받지 않음)

</br>

### UserDao의 의존관계

- 인터페이스를 통한 의존관계 → 관계가 느슨해지며 변화에 영향을 덜 받음
- 런타임 의존관계(오브젝트 의존관계): 런타임 시 오브젝트 사이에서 만들어지는 의존관계, 설계 시점의 의존관계가 실체화된 것 (모델링 시점의 의존관계 ❌)
- 의존 오브젝트: 프로그램이 시작되고 UserDao 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상 (실제 사용 대상인 오브젝트)
- 의존 관계 주입: 구체적인 의존 오브젝트와 이것을 사용하는 주체 오브젝트(클라이언트)를 **런타임** 시에 연결해주는 작업

**📌의존관계 주입의 3가지 조건**
<aside>

- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 **인터페이스에만 의존**해야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 **제3의 존재가 결정**(DaoFactory, 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등)한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.
</aside>

</br>

### UserDao의 의존관계 주입

```java
// 🚨 설계 시점에서 구체적인 클래스의 존재를 알고 있음
// -> 이후 DaoFactory를 만든 것 (DI 컨테이너)
public UserDao() {
		connectionMaker = new DConnectionMaker();
}
```

- DI 컨테이너는 자신이 결정한 의존관계를 맺어줄 클래스의 오브젝트를 만들고 이 생성자의 파라미터로 오브젝트의 레퍼런스를 전달
- DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고 자신은 수동적으로 주입받는 오브젝트를 사용
  - 스프링 컨테이너의 IoC는 주로 의존관계 주입 또는 DI에 초점이 맞추어져 있음

```java
public class UserDao {
      private ConnectionMaker connectionMaker;

      public void setConnectionMaker(ConnectionMaker connectionMaker) {
          this.connectionMaker = connectionMaker;
      }
  }
```

</br>

### 의존관계 검색과 주입

- 의존관계 검색(DL): 의존관계를 맺는 방법이 외부로부터의 주입이 아니라 스스로 검색을 이용
  - 외부 컨테이너: 런타임 시 의존관계를 맺을 오브젝트를 결정하고 오브젝트의 생성 작업을 함
  - 가져올 때는 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청

```java
// 애플리케이션 컨텍스트까지 일반화해서 생각해보면 -> 검색이라고 볼 수 있음
// DaoFactory를 이용하는 생성자
public UserDao() {
        DaoFactory daoFactory = new DaoFactory();
        this.connectionMaker = daoFactory.connectionMaker();
}
```

```java
// 의존관계 검색을 이용하는 UserDao 생성자

public UserDao() {
        AnnotationConfigAppcationContext context = new AnnotationConfigAppcationContext(DaoFactory.class);
        this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

- 코드 안에 오브젝트 팩토리 클래스나 스프링 API가 나타남
  - 애플리케이션 컴포넌트가 성격이 다른 오브젝트(컨테이너)에 의존해서 바람직하지 않음
- 스프링의 IoC와 DI 컨테이너를 적용해도 애플리케이션의 기동 시점에서 적어도 한 번은 의존관계 검색을 사용해 오브젝트를 가져와야 함
- 의존관계 검색 방식에서 검색하는 오브젝트는 스프링 빈일 필요가 없음
- 의존관계 주입 방식에서는 **UserDao도 빈 오브젝트**여야함
  - 컨테이너가 UserDao에 오브젝트를 주입하려면 UserDao에 대한 생성과 초기화 권한이 필요하기 때문

</br>

## ◾ 의존관계 주입의 응용

### 기능 구현의 교환

- ConnectionMaker의 다양한 활용이 가능

```java
@Bean
public ConnectionMaker connectionMaker() {
		return new LocalDBConnectionMaker(); // 개발용
		// return new ProductionDBConnectionMaker(); // 운영용
}
```

</br>

### 부가 기능 추가

```java
public interface ConnectionMaker {
    Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

```java
public class CountingConnectionMaker implements ConnectionMaker{
    int counter = 0;
    private ConnectionMaker realConnectionMaker;

		// DI를 받음
    public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
        this.realConnectionMaker = realConnectionMaker;
    }

		// DAO가 DB 커넥션을 가져올 때마다 호출
		// 카운터도 함께 증가
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        counter++;
        return realConnectionMaker.makeConnection();
    }

    public int getCounter() {
        return counter;
    }
}
```

```java
@Configuration
public class CountingDaoFactory {
		@Bean
		public UserDao userDao() {
				return new UserDao(ConnectionMaker());
		}

		@Bean
		public ConnectionMaker connectionMaker() {
				return new CountingConnectionMaker(realConnectionMaker());
		}

		@Bean
		public ConnectionMaker realConnectionMaker() {
				return new DConnectionMaker();
		}
}
```

```java
public class UserDaoConnectionCountingTest {

    public static void main(String[] args) throws SQLException, ClassNotFoundException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao userDao = context.getBean("userDaoWithCounting", UserDao.class);

				// DL 이용
        CountingConnectionMaker ccm = context.getBean("countingConnectionMaker", CountingConnectionMaker.class);
        System.out.println("Connection counter : " + ccm.getCounter());
    }
}
```

</br>

### 메소드를 이용한 의존관계 주입

- 수정자 메소드를 이용한 주입

```java
public class UserDao {
      private ConnectionMaker connectionMaker;

      public void setConnectionMaker(ConnectionMaker connectionMaker) {
          this.connectionMaker = connectionMaker;
      }
  }
```

```java
@Bean
public UserDao userDao() {
		UserDao userDao = new UserDao();
		userDao.setConnectionMaker(connectionMaker());
		return userDao;
}
```

- 일반 메소드를 이용한 주입 (수정자와 다르게 한번에 여러 개의 파라미터를 받을 수 있음)

</br>

# 7️⃣ XML을 이용한 설정

- 별도의 빌드 작업(컴파일)이 없음
- 환경이 달라져서 오브젝트의 관계가 바뀌는 경우 반영이 빠름
- 스키마나 DTD를 이용해서 쉽게 확인 가능

## ◾ XML 설정

- 빈의 이름: @Bean 메소드 이름, getBean()에서 가져올 때 사용
- 빈의 클래스: 빈 오브젝트를 어떤 클래스를 이용해서 만들지 결정
- 빈의 의존 오브젝트: 빈의 생성자, 수정자 메소드를 통해 의존 오브젝트를 넣어줌. 의존 오브젝트도 하나의 빈이므로 이름이 이을 것이고, 그 이름에 해당하는 메소드를 호출하여 의존 오브젝트를 가져옴 (의존하고 잇는 오브젝트가 없는 경우 생략 가능)

| 자바 코드 설정정보 | XML 설정정보            |
| ------------------ | ----------------------- |
| 빈 설정파일        | @Configuration          |
| 빈의 이름          | @Bean methodName()      |
| 빈의 클래스        | return new BeanClass(); |

</br>

### userDao() 전환

- 수정자 메소드 사용시 XML로 의존관계를 만들 때 편리
- name 에트리뷰트: DI에 사용할 수정자 메소드의 프로퍼티 이름
- ref 에트리뷰트: 주입할 오브젝트를 정의한 빈의 ID

</br>

## ◾ XML을 이용하는 애플리케이션 컨텍스트

- **GenericXmlApplicationContext**
- XML에서 빈의 의존관계 정보를 이용하는 IoC/DI 작업에는 GenericXmlApplicationContext 사용
- 클래스패스를 포함한 다양한 소스에서 설정파일을 읽어올 수 있음

```java
// 시작하는 /가 없어도 항상 루트에서부터 시작하는 클래스패스임
ApplicationContext ac = new GenericXmlApplicationContext("/applicationContext.xml");
```

- **ClassPathXmlApplicationContext**
- XML 파일을 클래스패스에서 가져올 때 사용할 수 있는 편리한 기능이 추가된 것

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("daoContext.xml", UserDao.class);
```

</br>

## ◾ DataSource 인터페이스로 변환

### DataSource 인터페이스 적용

- 자바가 제공하는 인터페이스 사용

```java
public class UserDao {
		private DataSource dataSource;

		public void setDataSource(DataSource dataSource) {
				this.dataSource = dataSource;
		}

		public void add(User user) throws SQLException {
				Connection c = dataSource.getConnection();
				...
		}
}
```

```java
@Bean
public DataSource dataSource() {
		SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

		dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
		dataSource.setUrl("jdbc:mysql://localhost/springbook");
		dataSource.setUsername("spring");
		dataSource.setPassword("book");

		return dataSource;
}
```

```java
@Bean
public UserDao userDao() {
		UserDao userDao = new UserDao();
		userDao.setDataSource(dataSource());
		return userDao;
}
```

</br>

### XML 설정 방식

- SimpleDriverDataSource 오브젝트는 단순 Class 타입의 오브젝트나 텍스트 값임

</br>

## ◾ 프로퍼티 값의 주입

### 값 주입

- 다른 빈 오브젝트의 레퍼런스가 아닌 단순 정보도 오브젝트를 초기화하는 과정에서 수정자 메소드에 넣을 수 있음 (ex. `dataSource.setDriverClass(com.mysql.jdbc.Driver.class);`)
  - 다른 빈 오브젝트의 레퍼런스(ref)가 아닌 단순 값(value) 주입 → 일종의 DI

```java
<property name="driverClass" value="com.mysql.jdbc.Driver"/>
<property name="url" value="jdbc:mysql://localhost/springbook"/>
<property name="username" value="spring"/>
<property name="password" value="book"/>
```

</br>

### value 값의 자동 변환

- 스프링이 프로퍼티의 값을 수정자 메소드의 파라미터 타입을 참고로 해서 적절한 형태로 변환
- XML에서 “com.mysql.jdbc.Driver”라는 텍스트 값을 오브젝트로 자동 변경함

```java
Class driverClass = Class.forName("com.mysql.jdbc.Driver");
dataSource.setDriverClass(driverClass);
```

</br>

# 8️⃣ 정리

- 관심사의 분리, 리팩토링
- 전략 패턴
- 개방 폐쇄 원칙
- 낮은 결합도, 높은 응집도
- 제어의 역전/IoC
- 싱글톤 레지스트리
- 의존관계 주입
- 생성자 주입과 수정자 주입
- XML 설정
