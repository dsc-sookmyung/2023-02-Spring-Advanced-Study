# 1️⃣ 사라진 SQLException

```java
public void deleteAll() throws SQLException {
    this.jdbcContext.executeSql("delete from users");
}
```

```java
// SQLException이 사라짐
public void deleteAll() {
		this.jdbcTemplate.update("delete from users");
}
```

## 초난감 예외처리

### 예외 블랙홀

- catch 블록을 써서 잡아낸 후 아무 처리도 하지 않음

```java
try {

    } catch(SQLException e){
}
```

- 시스템의 오류나 이상한 결과의 원인을 찾을 수 없음
- 화면에 출력한 경우도 예외를 처리한 것이 아님

```java
} catch(SQLException e){
    System.out.println(e);
}
```

```java
} catch(SQLException e){
    e.printStackTrace();
}
```

- 그나마 나은 처리 (좋은 처리는 아님)

```java
} catch(SQLException e){
    e.printStackTrace();
    System.exit(1);
}
```

- ✨ **예외를 무시하거나 잡아먹어 버리는 코드는 만들면 안됨**

### 무의미하고 무책임한 throws

- 실행 중 예외가 발생할 수 있다는 것인지, 습관적인 예외 처리인지 알 수 없음

```java
public void method1() throws Exception {
    method2();
}

public void method2() throws Exception {
    method3();
}

public void method3() throws Exception {
    ...
}
```

## 예외의 종류와 특징

- 가장 중요한 것은 체크 예외(명시적인 처리가 필요한 예외를 사용하고 다루는 방법)을 다루는 것
- **Error**
  - java.lang.Error 클래스의 서브 클래스
  - 주로 자바 VM에서 발생시키는 것으로 애플리케이션 코드에서 잡으려고 해도 대응방법이 없음
  - 개발자가 신경쓰지 않아도 됨

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/cffbc2f7-d0a1-4cb5-bca1-2978cc0c8da7/179e016b-b178-4e05-822f-ad6d49ca825d/Untitled.png)

- **Exception과 체크 예외**
  - java.lang.Exception 클래스와 그 서브 클래스로 개발자들이 사용함
    1. 체크 예외
       - Excepion 클래스의 서브 클래스, RuntimeException을 상속하지 않음
       - 반드시 예외 처리를 하는 코드를 함께 작성해야 함
    2. 언체크 예외
       - RuntimeException을 상속한 클래스 (Exception의 일종이지만 자바가 해당 Exception과 서브클래스는 특별하게 다룸)
- **RuntimeException과 언체크/런타임 예외**
  - 명시적인 예외처리를 강제하지 않음 (예외 처리 해도 됨)
  - 프로그램의 오류가 있을 때 발생하도록 의도된 것
  - NullPointerException, IllegalArgumentException
  - 코드에서 미리 조건을 체크하도록 주의 깊게 만든다면 피할 수 있음
  - 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것

## 예외처리 방법

### 예외 복구

- 예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것
- IOException: 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없다거나 다른 문제가 있어서 발생
  - 사용자가 다른 파일을 이용하도록 안내하여 해결 가능
- SQLException: 네트워크가 불안해서 가끔 접속이 안되는 시스템. 원격DB 서버 접속 실패해서 발생한 경우
  - 일정 시간 대기 후 접속 재시도
  - 정해진 횟수만큼 시도한 후에도 실패한다면 복구 포기

```java
int maxretry = MAX_RETRY;
while(maxretry -- >0) {
		try {
        ...     // 예외가 발생할 가능성이 있는 시도
        return; // 작업 성공
    catch(SomeException e) {
	      // 로그 출력. 정해진 시간만큼 대기
    finally {
        // 리소스 반납. 정리 작업
    }
    throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```

### 예외처리 회피

- 예외처리를 자신을 호출한 쪽으로 던짐
- throws문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것(rethrow)
- 콜백 오브젝트의 메소드는 SQLException에 대한 예외를 회피하고 템플릿 레벨에서 처리하도록 던져줌
- 콜백과 템플릿처럼 긴밀하게 역할을 분담하고 있는 관계가 아니라면 무책임한 회피가 될 수 있음
- 의도가 분명해야 함

```java
public void add() throws SQLException {
    // JDBC API
}
```

```java
public void add() throws SQLException {
		try {
        // JDBC API
    } catch(SQLException e) {
        // 로그 출력
        throw e;
    }
}
```

### 예외 전환

- 발생한 예외를 그대로 넘기지 않고 **적절한 예외로 전환**해 던짐

1. 내부에 발생한 예외를 분명한 의미를 가진 예외로 바꿔서 던지기 위해

- API가 발생하는 기술적인 로우레벨을 상황에 적합한 의미를 가진 예외로 변경하는 것
- 새로운 사용자 등록 시 같은 아이디의 사용자가 있어 DB 에러 발생
- SQLException의 정보를 해석해서 DuplicationUserIdException같은 예외로 바꾸어서 던짐
- 특정 기술 정보를 해석하는 코드를 비즈니스 로직을 담은 서비스 계층에 두는 것은 어색함
  → DAO 메소드에서 기술에 독립적이며 의미가 분명한 예외로 전환 후 던져줄 필요가 있음

```java
public void add(User user) throws DuplicateUserIdException, SQLException {
    try {
        // JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch(SQLException e) {
        // ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
        if (e.QetErrorCode() == MysQIErrorNumbers.ER_DUP_ENTRY)
            throw new DuplicateUserIdException();
    } else
        throw e; // 그 외의 경우는 SQLException 그대로
```

- **중첩 예외**: 전환하는 예외에 원래 발생한 예외를 담는 것
- 생성자나 initCause() 메소드로 근본 원인이 되는 예외를 넣어주면 됨

```java
catch(SQLException e) {
		...
		throw DuplicateUserIdException(e);
}
```

```java
catch(SQLException e) {
		...
		throw DuplicateUserIdException().initCause(e);
}
```

- 예외를 처리하기 쉽고 단순하게 만들기 위해 **포장**함
- 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용
- EJBException
  - EJB 컴포넌트 코드에서 발생하는 대부분의 체크 예외는 비즈니스 로직으로 볼 때 의미있는 예외이거나 복구 가능한 예외가 아님
  - EJBException으로 포장해서 던지는 것이 나음
  - RuntimeException 클래스를 상속한 런타임 예외이므로 트랜젝션 자동 롤백 가능)
  - 잡아도 복구할만한 방법이 없음

```java
try {
    OrderHome orderHome = EJBHomeFactorY.getlnstance().getOrderHome();
    Order order = orderHome.findByPrimaryKey(Integer id);
} catch (NamingException ne) (
    throw new EJBException(ne);
} catch (SQLException se) (
    throw new EJBException(se);
} catch (RemoteException re ) (
    throw new EJBException(re)
}
```

- 애플리케이션 로직상에서 예외조건이 발견되거나 예외상황이 발생할 수 있음
  → 체크 예외를 사용하는 것이 적절

## 예외처리 전략

### 런타임 예외의 보편화

- 일반적으로 체크 예외 - 일반적 예외/ 언체크 예외 - 시스템 장애나 프로그램상의 오류에 사용
- 독립형 어플리케이션(ex. 애플릿, AWT, 스윙)
  - 통제 불가능한 시스템 예외라도 애플리케이션의 작업이 중단되지 않게 해주고 상황을 최대한 복구해야함
- 자바 엔터프라이즈 서버 환경
  - 수많은 사용자 동시 요청을 처리해야 하며 각 요청은 독립적 작업임
  - 하나의 요청 처리중 예외 발생시 해당 작업만 중단하면 됨
  - 예외 발생시 사용자와 커뮤니케이션 하면서 복구할 수 있는 수단이 없음
  - 예외 상황을 미리 파악하고, 예외가 발생치 않도록 차단하는 게 좋음
  - 빨리 요청의 작업 취소 후, 서버 관리자나 개발자에게 통보해야 함
  - 체크예외는 점점 사용도가 떨어지고 있으며 대부분 런타임 예외로 처리하는 경향 증가
  - 언체크라도 언제든지 예외를 catch로 잡을 수 있지만 대개 복구 불가능한 상황이므로 API 차원에서 런타임 에러로 처리

### add() 메소드의 예외처리

- SQLException, DuplicateUserIdException 존재
- SQLException은 런타임 예외로 포장해 던지는 것이 나음
- DuplicateUserIdException를 반드시 체크 예외로 두어야 하는 것은 아니며 런타임 예외로 만드는 것이 나음

```java
public class DuplicateUserldException extends RuntimeException {
		public DuplicateUserldException(Throwable cause) {
		    super (cause);
		}
}
```

```java
public void add(User user) throws DuplicateUserIdException {
		try {
		    // JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    } catch(SQLException e) { // 언체크 예외가 됨, throws에 포함시킬 필요가 없음
        if (e.QetErrorCode() == MysQIErrorNumbers.ER_DUP_ENTRY)
		        throw new DuplicateUserIdException(e); // 예외 전환
    } else
        throw new RuntimeException(e); // 예외 포장
```

### 애플리케이션 예외

- 런타임 예외 중심 전략은 낙관적인 예외처리 기법이라고 할 수 있음
- 복구할 수 있는 예외는 없다는 가정
- 어차피 시스템 레벨에서 알아서 처리할 것이며 꼭 필요한 경우는 런타임 예외라도 잡아서 복구할 수 있으므로 문제될 것이 없다는 낙관적인 태도를 기반으로 함
- 일단 잡고 보도록 강제하는 체크 예외의 비관적인 접근 방법과 대비됨
- 애플리케이션 예외
- 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고 반드시 catch해서 조치를 취하도록 요구하는 예외
  - ex) 사용자가 요청한 금액을 은행계좌에서 출금하는 기능을 가진 메소드
  - 설계 방법1. 정상적인 출금처리를 했을 경우와 잔고 부족이 발생했을 경우에 각각 다른 종류의 리턴 값을 돌려줌
  - 설계 방법2. 정상적 흐름을 따르는 코드는 그대로 두고, 잔고 부족과 같은 예외상황에서는 비즈니스적인 의미를 띈 예외를 던지도록 만드는 것 (InsufficientBalenceException: 잔고 부족인 경우)
  ```java
  try {
  		BigDecimal balance = account .withdraw(amount);
      ...
      // 정상적인 처리 결과를 출력하도록 진행
  catch(InsufficientBalanceException e) { // 체크 예외
      // InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
      BigDecimal avai lFunds = e.getAvailFunds();
      ...
      // 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
  }
  ```

## SQLException은 어떻게 됐나?

- SQLException은 복구 가능한 예외가 아님
- 가능한 의미있는 언체크/런타임 예외로 전환해서 던지는 것이 나음
- 스프링은 JdbcTemplate에서 이와 같은 예외처리 전략을 따름
  - ✨ **SQLException을 런타임 예외 DataAccessException으로 포장해 던짐**
  - 스프링을 사용하는 측은 꼭 필요한 경우에만 catch 해서 처리하면 됨

# 2️⃣ 예외 전환

- 예외 전환 목적1) 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄여주는 것
- 예외 전환 목적2) 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바뀌서 던져주는 것
- 스프링 JdbcTemplate이 던지는 DataAccessException의 목적
  - SQLException을 런타임 예외로 포장 → 대부분 복구 불가능한 예외를 애플리케이션 레벨에서 신경쓰지 않도록 해줌
  - SQLException에서 다루기 힘든 상세한 예외정보를 의미있고 일관성 있는 예외로 전환해서 추상화해줌

## JDBC의 한계

- 자바를 이용해 DB에 접근하는 방법을 추상화된 API 형태로 정의해놓고 각 DB 업체가 JDBC 표준을 따라 만들어진 드라이버를 제공하게 해줌
- DB를 자유롭게 변경해서 사용할 수 있는 유연한 코드를 보장해주지는 못함

### 비표준 SQL

- 대부분의 DB가 비표준 문법과 기능이 있고, 해당 비표준이 매우 폭넒게 사용됨
- DB의 특별한 기능을 사용하거나 최적화된 SQL을 만들 때 유용
  - ex) 최적화 기법, 페이지 처리를 위한 로우 시작위치와 개수 지정, 쿼리에 조건 포함
- 작성된 비표준 SQL은 DAO 코드에 들어가고 해당 DAO는 특정 DB에 종속적인 코드가 됨
- DAO를 DB별로 만들어 사용 또는 SQL을 외부에 독립시켜서 바꿔 쓸 수 있게 하는 것이 현실적

### 호환성 없는 SQLException의 DB 에러정보

- DB마다 에러의 종류와 원인도 다름
- JDBC는 그 제각각 예외를 SQLException 한 곳에 모두 담아서 처리
- 예외가 발생한 원인은 SQLException 안의 에러 코드와 SQL 상태정보를 참조해야 함
- DB마다 에러 코드가 다름
- DB의 JDBC 드라이버에서 SQLException을 담을 상태 코드를 명확하게 만들어주지 않음
- 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 SQLException만으로 DB에 독립적인 유연한 코드를 작성하는 것이 불가능

## DB 에러 코드 매핑을 통한 전환

- SQLException의 비표준 에러 코드와 SQL 상태 정보에 대한 해결책
- 스프링은 DataAccessException 이라는 SQLException을 대체할 수 있는 런타임 예외를 정의
- DataAccessException의 서브클래스로 세분화된 예외 클래스들을 정의
  - BadSqlGrammerException: SQL문법이 틀리면 발생
  - DataAccessResourceFailureException: DB 커넥션을 가져오지 못했을 때
  - DataIntegrityViolationException: 데이터 제약조건을 위배했거나 일관성을 지키지 않는 작업을 수행했을 때
  - DuplicationKeyException: 그 중 중복 키 때문에 발생한 경우
- JdbcTemplate은 SQLException을 런타임 예외인 DataAccessException으로 포장하는 것이 아니라 DB의 에러 코드를 DataAccessException 계층 구조의 클래스 중 하나로 매핑
- 전환되는 JdbcTemplate에서 던지는 예외는 모두 DataAccessException의 서브클래스 타입
  - DB가 달라져도 같은 종류의 에러라면 동일한 예외를 받을 수 있음

```java
public void add() throws DuplicateKeyException {
		// JdbcTemplate을 이용해 User를 add 하는 코드
}
```

- 직접 정의한 예외를 발생시키고 싶은 경우

```java
public void add() throws DuplicateUserIdException {
		try {
		    // jdbcTempate을 이용해 User를 add 하는 코드
    }
    catch(DuplicateKeyException e) {
        throw new DuplicateUserIdException(e); // 원인이 되는 예외를 중첩하는 것이 좋음
    }
}
```

- SQLException의 서브클래스이므로 체크 예외라는 점과 그 예외를 세분화하는 기준이 SQL 상태 정보를 이용한다는 점에서 문제가 있으나 현재 가장 이상적인 방법임

## DAO **인터페이스와 DataAccessException 계층구조**

- DataAccessException은 JDBC 예외 뿐만 아니라 자바 데이터 엑세스 기술에서 발생하는 예외에도 적용됨 (ex. JDO, JPA, 하이버네이트, myBatis)
  - 데이터 엑세스 기술의 종류와 상관없이 일관된 예외가 발생하도록 함
  - 데이터 액세스 기술에 독립적인 추상화된 예외를 제공

### DAO 인터페이스와 구현의 분리

- DAO를 따로 만들어서 사용하는 이유
  1. 데이터 엑세스 로직을 담은 코드를 성격이 다른 코드에서 분리해놓기 위해
  2. 분리된 DAO는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있게 만들기 위함
     - DAO를 사용하는 쪽에서 DAO가 내부에서 사용하는 데이터 기술을 신경 쓰지 않아도 됨
- 🚨 DAO의 사용 기술과 구현 코드는 전략 패턴과 DI를 통해 DAO를 사용하는 클라이언트에게 감출 수 있으나, 메소드 선언에 나타나는 **예외정보**가 문제가 될 수 있음

```java
public interface UserDao {
    public void add(User user); //이렇게 선언하는 것이 가능한가?
}
```

- 🚨 예외가 기술 독립적으로 인터페이스 선언 하지 못하도록 되어 있음

```java
public void add(User user) throws PersistentException // JPA
public void add(User user) throws PersistentException // Hibernate
public void add(User user) throws PersistentException // JDO
```

- 가장 단순한 해결 방법은 throw Exception으로 선언하는 것 → 무책임한 선언
- 대부분의 데이터 액세스 예외는 애플리케이션에서 복구 불가능하거나 할 필요가 없지만 모든 예외를 다 무시해야 하는 것은 아님
- 애플리케이션에서는 사용하지 않더라도 시스템 레벨에서 데이터 엑세스 예외를 의미 있게 분류할 필요가 있음
- 🚨 데이터 엑세스 기술이 달라지면 같은 상황에서도 다른 종류의 예외가 던져진다는 것
  - DAO를 사용하는 클라이언트 입장에서는 DAO의 사용 기술에 따라서 예외 처리 방법이 달라져야 함
    → 클라이언트가 DAO 기술에 의존적

### 데이터 액세스 예외 추상화와 DataAccessException 계층 구조

- DataAccessException은 자바의 주요 데이터 엑세스 기술에서 발생할 수 있는 대부분의 예외를 추상화함
  - 일부 기술에서만 공통적으로 나타나는 예외를 포함해서 데이터 액세스 기술에서 발생 가능한 대부분의 계층구조로 분류해둠
  - InvalidDataAccessResourceUsageException: 데이터 엑세스 기술을 부정확하게 사용
  - 하이버네이트에서는 HibernateQueryException/ TypeMismatchDataAccessException
  - JDBC에는 BadSqlGrammerException
- JDO, JPA, 하이버네이트처럼 오브젝트/엔티티 단위로 정보를 업데이트하는 경우에는 낙관적인 락킹 발생 가능

  - 스프링의 예외 전환 방법을 사용하면 기술에 상관없이 DataAccessException의 서브크래스인 ObjectOptimisticLockingFailureException으로 통일

- 인터페이스 사용, 런타임 예외 전환과 함께 DataAccessException 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO를 만들수 있다

## 기술에 독립적인 UserDao 만들기

### 인터페이스 적용

- UserDao 클래스를 인터페이스와 구현으로 분리
- 인터페이스 명명법
  1. 이름 앞에 'I'라는 접두어를 붙이는 방법
  2. 인터페이스의 이름을 단순하게 하고 구현 클래스는 각각의 특징을 따르는 이름을 붙임

```java
public interface UserDao {
    void add(User user);
    User get(String id);
    List<User> getAll();
    void deleteAll();
    int getCount();
}
```

```java
public class UserDaoJdbc implements UserDao {}
```

### 테스트 보완

```java
public class UserDaoTest {
		@Autowired
    private UserDao dao; // UserDaoJdbc로 변경해야 하나?
}
```

- 변경하지 않아도 됨
- @Autowired는 스프링의 컨텍스트 내에서 정의된 빈 중에서 인스턴스 변수에 주입 가능한 타입의 빈을 찾아줌
- UserDao는 UserDAoJdbc가 구현한 인터페이스이므로 UserDaoTest의 dao 변수에 UserDaoJdbc클래스로 정의된 빈을 넣는데 아무런 문제가 없음
- 구현 기술에 상관없이 DAO의 기능이 동작하는데만 관심이 있다면, UserDao 인터페이스로 받아서 테스트하는 것이 나음
- 특정 기술을 사용한 UserDao의 구현 내용에 관심을 가지고 테스트하려면 @Autowired로 DI 받을때 UserDaoJdbc나 UserDaoHibernate 같이 특정 타입을 사용하는 것이 나음

UserDao 인터페이스와 구현의 분리

```java
@Test(expected=DataAccessException.class) // 예외가 발생하면 성공
public void duplicateKey() {
    dao.deleteAll();

    dao.add(user1);
    dao.add(user1); // 예외가 발생해야 함, DataAccessException 예외 중의 하나가 던져져야 함
}
```

- DataAccessException의 서브클래스인 DataIntegrityViolationException의 한 종류인DuplicationKeyException이 발생

### DataAccessException 활용 시 주의사항

- 하위 예외인 DuplicationKeyException은 JDBC를 사용하는 경우에만 발생하는 것으로 DB종류나 데이터 엑세스 기술에 상관없이 동일하게 발생하지 않음
- JDBC는 SQLException에 담긴 DB 에러 코드를 바로 해석하지만 JPA, 하이버네이트, JDO 등에서는 각 기술이 재정의한 예외를 가져와 최종적으로 DataAccessException으로 변환하는데 DB의 에러 코드와 달리 이런 예외들은 세분화되어 있지 않음
- DataAccessException이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만 근본적인 한계 때문에 완벽하다고 기대할 수 없음 → 사용에 주의 필요
- DAO에서 사용하는 기술의 종류와 상관없이 동일한 예외를 얻고 싶다면 DuplicatedUserIdException처럼 직접 예외를 정의해두고 각 DAO의 add() 메소드에서 좀 더 상세한 예외 전환을 해줄 필요가 있음
- 스프링은 SQLException을 DataAccessException으로 변환하는 다양한 방법을 제공
  - 보편적인 방법은 DB 에러 코드를 이용하는 것

```java
public class UserDaoTest {
		@Autowired UserDao dao;
		@Autowired DataSource dataSource;
```

```java
@Test
public void sqlExceptionTranslate() {
    dao.deleteAll();

    try {
        dao.add(user1);
        dao.add(user1);
    } catch(DuplicateKeyException ex) {
				// 중첩되어 있는 SQLException을 가져올 수 있음
        SQLException sqlEx = (SQLException) ex.getRootCause();
				// 코드를 이용한 SQLException의 전환
				// SQLErrorCodeSQLExceptionTranslator는 에러 코드 변환에 필요한 DB의 종류를 알아내기 위해 DataSource를 필요로 함
        SQLExceptionTranslator set = new SQLErrorCodeSQLExceptionTranslator(this.dataSource);
				// 에러 메시지 만들 때 사용하는 정보이므로 null로 넣어도 됨
				// translate(): SQLException을 DataAccessException 타입의 예외로 변환
        assertThat(set.translate(null, null, sqlEx),is(DuplicateKeyException.class));
    }
}
```

# 3️⃣ 정리

- 예외를 잡아서 아무런 조취를 취하지 않거나 의미 없는 throws 선언을 남발하는 것은 위험함
- 예외는 복구하거나, 예외처리 오브젝트로 의도적으로 전달하거나, 적절한 예외로 전환해야 함
- 좀 더 의미 있는 예외로 변경하거나, 불필요한 catch/throws를 피하기 위해 런타임 예외로 포장하는 두 가지 방법의 예외 전환이 있음
- 복구할 수 없는 예외는 가능한 빨리 런타임 예외로 전환하는 것이 바람직함
- 애플리케이션의 로직을 담기 위한 예외 → 체크 예외로 만들기
- JDBC의 SQLException은 대부분 복구할 수 없는 예외 → 런타임 예외로 포장
- SQLExcetion의 에러 코드 → DB에 종속, DB에 독립적인 예외로 전환될 필요가 있음
- 스프링은 DataAccessException을 통해 DB에 독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공
- DAO를 데이터 엑세스 기술에서 독립시키려면 인터페이스 도입과 런타임 예외 전환, 기술에 독립적인 추상화된 예외로 전환이 필요함
