# 3. 예외처리

1. 사라진 SQLException

1) 초난감 예외처리

- 예외 블랙홀

```java
try { ... } catch(SQLException e) { }
```

예외가 발생하면 그것을 catch 블록을 써서 잡아내는 것까지는 좋은데 그리고 아무것도 하지 않고 별문제 없는 것처럼 넘어가 버리는 건 위험하다. 

→ 프로그램 실행 중에 어디선가 오류가 있어서 예외가 발생했는데 그것을 무시하고 계속 진행해버리기 때문이다. 

→ 결국 발생한 예외로 인해 어떤 기능이 비정상적으로 동작하거나, 메모리나 리소스가 소진되거나, 예상치 못한 다른 문제를 일으킬 것이다. 더 큰 문제는 그 시스템 오류나 이상한 결과의 원인이 무엇인지 찾아내기가 매우 힘들다는 점이다. 

```java
catch (SQLException e) { System.out.println(e); }
```

```java
catch (SQLException e) { e.printStackTrace(); }
```

catch 블록을 이용해 화면에 메시지를 출력한 것은 예외를 처리한 게 아니다.

→ 운영서버에 올라가면, 콘솔 로그를 누군가가 계속 모니터링하지 않는 한 이 예외 코드는 심각한 폭탄으로 남아 있을 것이다. 

```java
catch (SQLException e) { e.printStackTrace(); System.exit(1); }
```

굳이 예외를 잡아서 뭔가 조치를 취할 방법이 없다면 잡지 말아야 한다. 메소드에 throws SQLException 을 선언해서 메소드 밖으로 던지고 자신을 호출한 코드에 예외처리 책임을 전가해버려라.

모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 한다. 

- 무의미하고 무책임한 throws

```java
public void method1() throws Exception { method2(); ... } 호출 public void method2() throws Exception { method3(); ... } 호출 public void method3() throws Exception { ... }
```

API 등에서 발생하는 예외를 일일이 catch 하기도 귀찮고, 별 필요도 없으며 매번 정확하게 예외 이름을 적어서 선언하기도 귀찮으니 아예 throws Exception 이라는, 모든 예외를 무조건 던져버리는 선언을 모든 메소드에 기계적으로 넣는 것이다.

→ 메소드 선언에서는 의미 있는 정보를 얻을 수 없다. 정말 무엇인가 실행 중에 예외 적인 상황이 발생할 수 있다는 것인지, 아니면 그냥 습관적으로 복사해서 붙여놓은 것인지 알 수가 없다. 

→ 결국 이런 메소드를 사용하는 메소드에서도 역시 throws Exception 을 따라서 붙이는 수밖에 없다. 결과적으로 적절한 처리를 통해 복구될 수 있는 예외상황도 제대로 다룰 수 있는 기회를 박탈당한다.

2) 예외의 종류와 특징

체크 예외(checked exception)는 불리는 명시적인 처리가 필요한 예외를 사용하고 다루는 방법이다.

자바에서 throw 를 통해 발생시킬 수 있는 예외는 크게 세 가지가 있다.

- Error

java.lang.Error 클래스의 서브클래스들이다. 에러는 시스템에 뭔가 비정상 적인 상황이 발생했을 경우에 사용된다. 그래서 주로 자바 VM에서 발생시키는 것이고 애플리케이션 코드에서 잡으려고 하면 안 된다. OutOfMemoryError 나 ThreadDeath 같은 에러는 catch 블록으로 잡아봤자 아무런 대응 방법이 없기 때문이다. 

→ 시스템 레벨에서 특별한 작업을 하는 게 아니라면 애플리케이션에서는 이런 에러에 대한 처리는 신경 쓰지 않아도 된다.

- Exception과 체크 예외

java.lang.Exception 클래스와 그 서브클래스로 정의되는 예외들은 에러와 달리 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용된다. 

** 체크예외 

Exception 클래스의 서브클래스이면서 RuntimeException 클래스를 상속하지 않은 것들이다.

**언체크예외

Exception 클래스의 서브클래스이면서 RuntimeException 을 상속한 클래스들을 말한다. 


일반적으로 예외라고 하면 Exception 클래스의 서브클래스 중에서 RuntimeException 을 상속하지 않은 것만을 말하는 체크 예외라고 생각해도 된다.

 체크 예외가 발생할 수 있는 메소드를 사용할 경우 반드시 예외를 처리하는 코드를 함께 작성해야 한다. 사용할 메소드가 체크 예외를 던진다면 이를 catch 문으로 잡든지, 아니면 다시 throws 를 정의해서 메소드 밖으로 던져야 한다. 그렇지 않으면 컴파일 에러가 발생한다.

- RuntimeException과 언체크/런타임 예외

java.lang.RuntimeException 클래스를 상속한 예외들은 명시적인 예외처리를 강제 하지 않기 때문에 언체크 예외라고 불린다. 또는 대표 클래스 이름을 따서 런타임 예외라고도 한다. 

에러와 마찬가지로 이 런타임 예외는 catch 문으로 잡거나 throws 로 선언하지 않아도 된다. 런타임 예외는 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들이다.

- NullPointerException

오브젝트를 할당하지 않은 레퍼런스 변수를 사용하려고 시도했을 때 발생

- IllegalArgumentException

허용되지 않는 값을 사용해서 메소드를 호출할 때 발생

 피할 수 있지만 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것이 런타임 예외다. 

→ 따라서 런타임 예외는 예상 하지 못했던 예외상황에서 발생하는 게 아니기 때문에 굳이 catch 나 throws 를 사용하지 않아도 되도록 만든 것이다.

3) 예외처리 방법

다음은 예외를 처리하는 일반적인 방법이다. 

- 예외 복구

첫 번째 예외처리 방법은 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것이다.

IOException 에러 메시지가 사용자에게 그냥 던져지는 것은 예외 복구라고 볼 수 없다. 

→ 예외가 처리됐으면 비록 기능적으로는 사용자에게 예외상황으로 비쳐도 애플리케이션에 서는 정상적으로 설계된 흐름을 따라 진행돼야 한다.

예외처리 코드를 강제하는 체크 예외들은 예외를 어떤 식으로든 복구할 가능성이 있는 경우에 사용한다. 

```java
maxretry = MAX_RETRY; while(maxretry -- > 0) { try { ...  return; } catch(SomeException e) { } finally { } } throw new RetryFailedException(); 

- 예외처리 회피

예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것이다. 예외를 자신이 처리하지 않고 회피하는 방법이다. 

throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 rethrow 것이다.

```java
public void add() throws SQLException {  }
```

```java
public void add() throws SQLException { try {  } catch(SQLException e) {  } }
```

콜백 오브젝트의 메소드는 모두 throws SQLException 이 붙어 있다. SQLException 을 처리하는 일은 콜백 오브젝 트의 역할이 아니라고 보기 때문이다. 콜백 오브젝트의 메소드는 SQLException 에 대한 예외를 회피하고 템플릿 레벨에서 처리하도록 던져준다.

하지만 콜백과 템플릿처럼 긴밀하게 역할을 분담하고 있는 관계가 아니라면 자신의 코드에서 발생하는 예외를 그냥 던져버리는 건 무책임한 책임회피일 수 있다.

→ 예외를 회피하는 것은 예외를 복구하는 것처럼 의도가 분명해야 한다. 콜백/템플 릿처럼 긴밀한 관계에 있는 다른 오브젝트에게 예외처리 책임을 분명히 지게 하거나, 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분명한 확신이 있어야 한다.

- 예외 전환

예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것이다. 하지만 예외 회피와 달리, 발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다는 특징이 있다. 

**예외 전환의 목적

 (1) 내부에서 발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우에, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서다. API가 발생하는 기술적인 로우레벨을 상황에 적합한 의미를 가진 예외로 변경하는 것이다.

```java
public void add(User user) throws DuplicateUserIdException, SQLException { try {  } catch(SQLException e) { if (e.getErrorCode() = = MysqlErrorNumbers.ER_DUP_ENTRY) throw DuplicateUserIdException(); else throw e;  } }
```

보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외 nested exception 로 만드는 것이 좋다. 중첩 예외는 getCause() 메소드를 이용해서 처음 발생한 예외가 무엇인지 확인할 수 있다. 중첩 예외는 새로운 예외를 만들면서 생성자나 initCause() 메소드로 근본 원인이 되는 예외를 넣어주면 된다.

```java
catch(SQLException e) { ... throw DuplicateUserIdException(e); 
```}

```java
catch(SQLException e) { ... throw DuplicateUserIdException().initCause(e);
```}

(2) 전환 방법은 예외를 처리하기 쉽고 단순하게 만들기 위해 포장 wrap 하는 것이다. 중첩 예외를 이용해 새로운 예외를 만들고 원인 cause 이 되는 예외를 내부에 담아서 던지는 방식은 같다. 하지만 의미를 명확하게 하려고 다른 예외로 전환하는 것이 아니다. 주로 예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용한다.

```java
try { OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome(); Order order = orderHome.findByPrimaryKey(Integer id); } catch (NamingException ne) { throw new EJBException(ne); } catch (SQLException se) { throw new EJBException(se); } catch (RemoteException re) { throw new EJBException(re); }
```

이렇게 런타임 예외로 만들어서 전달하면 EJB는 이를 시스템 익셉션으로 인식하고 트랜잭션을 자동으로 롤백해준다. 런타임 예외이기 때문에 EJB 컴포넌트를 사용하는 다른 EJB나 클라이 언트에서 일일이 예외를 잡거나 다시 던지는 수고를 할 필요가 없다. 이런 예외는 잡아도 복구할 만한 방법이 없기 때문이다.

 반대로 애플리케이션 로직상에서 예외조건이 발견되거나 예외상황이 발생할 수도 있다. 이런 것은 API가 던지는 예외가 아니라 애플리케이션 코드에서 의도적으로 던지는 예외다. 이때는 체크 예외를 사용하는 것이 적절하다. 비즈니스적인 의미가 있는 예외는 이에 대한 적절한 대응이나 복구 작업이 필요하기 때문이다. 

일반적으로 체크 예외를 계속 throws를 사용해 넘기는 건 무의미하다. 메소드 선언은 지저분해지고 아무런 장점이 없다. 어차피 복구가 불가능한 예외라면 가능한 한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 throws 선언이 들어가지 않도록 해줘야 한다.

4) 예외처리 전략

- 런타임 예외의 보편화

일반적으로는 체크 예외가 일반적인 예외를 다루고, 언체크 예외는 시스템 장애나 프로 그램상의 오류에 사용한다고 했다. 문제는 체크 예외는 복구할 가능성이 조금이라도 있는, 말 그대로 예외적인 상황이기 때문에 자바는 이를 처리하는 catch 블록이나 throws 선언을 강제하고 있다는 점이다.

→ 자바의 환경이 서버로 이동하면서 체크 예외의 활용도와 가치는 점점 떨어지고 있다. 자칫하면 throws Exception 으로 점철된 아무런 의미도 없는 메소드들을 낳을 뿐이다. 그래서 대응이 불가능한 체크 예외라면 빨리 런타임 예외로 전환해서 던지는 게 낫다.

- add() 메소드의 예외처리

DuplicatedUserIdException 은 충분히 복구 가능한 예외이므로 add() 메소드를 사용하는 쪽에서 잡아서 대응할 수 있다. 하지만 SQLException 은 대부분 복구 불가능한 예외이므로 잡아봤자 처리할 것도 없고, 결국 throws 를 타고 계속 앞으로 전달되다가 애플리케이션 밖으로 던져질 것이다. 그럴 바에는 그냥 런타임 예외로 포장해 던져버려서 그 밖의 메소드들이 신경 쓰지 않게 해주는 편이 낫다.

D u p l i c a t e d U s e r I d E x c e p t i o n 도 굳이 체크 예외로 둬야 하는 것은 아니 다. D u p l i c a t e d U s e r I d E x c e p t i o n 처럼 의미 있는 예외는 a d d ( ) 메소드를 바로 호출한 오브젝트 대신 더 앞단의 오브젝트에서 다룰 수도 있다. 어디에서든 D u p l i c a t e d U s e r I d E x c e p t i o n 을 잡아서 처리할 수 있다면 굳이 체크 예외로 만들지 않고 런타임 예외로 만드는 게 낫다. 대신 add() 메소드는 명시적으로 DuplicatedUserIdException 을 던진다고 선언해야 한다. 그래야 add() 메소드를 사용하는 코드를 만드는 개발자에게 의미 있는 정보를 전달해줄 수 있다. 런타임 예외도 throws 로 선언할 수 있으니 문제 될 것은 없다.

→ 런타임 예외로 만들었기 때문에 사용에 더 주의를 기울일 필요도 있다. 컴파일러가 예외처리를 강제하지 않으므로 신경 쓰지 않으면 예외상황을 충분히 고려하지 않을 수도 있기 때문 이다. 런타임 예외를 사용하는 경우에는 API 문서나 레퍼런스 문서 등을 통해, 메소드를 사용할 때 발생할 수 있는 예외의 종류와 원인, 활용 방법을 자세히 설명해두어야 한다.

- 애플리케이션 예외

애플리케이션 예외란, 시스템 또는 외부의 예외상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch 해서 무엇인가 조치를 취하도록 요구하는 예외이다.

예금을 인출해서 처리하는 코드를 정상 흐름으로 만들어두고, 잔고 부족을 애플리케이션 예외로 만들어 처리하도록 만든 코드다. 애플리케이션 예외인 InsufficientBalanceException 을 만들 때는 예외상황에 대한 상세한 정보를 담고 있도록 설계할 필요가 있다. 잔고가 부족한 경우라면 현재 인출 가능한 최대 금액은 얼마인지를 확인해서 예외 정보에 넣어준다면 좋을 것이다.

```java
try { BigDecimal balance = account.withdraw(amount); ...  } catch(InsufficientBalanceException e) { }
```

5) SQLException은 어떻게 됐나?

99%의 SQLException 은 코드 레벨에서는 복구할 방법이 없다. 프로그램의 오류 또는 개발자의 부주의 때문에 발생하는 경우이거나, 통제할 수 없는 외부상황 때문에 발생하는 것이다. 예를 들어 SQL 문법이 틀렸거나, 제약조건을 위반했거나, DB 서버가 다운됐다거나, 네트워크가 불안정하거나, DB 커넥션 풀이 꽉 차서 DB 커넥션을 가져올 수 없는 경우 등이다. 시스템의 예외라면 당연히 애플리케이션 레벨에서 복구할 방법이 없다. 관리자나 개발자에게 빨리 예외가 발생했다는 사실이 알려지도록 전달하는 방법밖에는 없다. 마찬가 지로 애플리케이션 코드의 버그나 미처 다루지 않았던 범위를 벗어난 값 때문에 발생한 예외도 역시 복구할 방법이 없다.

→ 따라서 예외처리 전략을 적용해야 한다. 필요도 없는 기계적인 throws 선언이 등장하도록 방치하지 말고 가능한 한빨리 언체크/런타임 예외로 전환해줘야 한다. 스프링의 JdbcTemplate 은 바로 이 예외처리 전략을 따르고 있다. JdbcTemplate 템플 릿과 콜백 안에서 발생하는 모든 SQLException 을 런타임 예외인 DataAccessException 으로 포장해서 던져준다. 따라서 JdbcTemplate 을 사용하는 UserDao 메소드에선 꼭 필요한 경우에만 런타임 예외인 DataAccessException 을 잡아서 처리하면 되고 그 외의 경우에는 무시해도 된다. 

1. 예외 전환

1) JDBC의 한계

JDBC는 자바 표준 JDK에서도 가장 많이 사용되는 기능 중의 하나지만, DB 종류에 상관없이 사용할 수 있는 데이터 액세스 코드를 작성하는 일은 쉽지 않다. 표준화된 JDBC API가 DB 프로그램 개발 방법을 학습하는 부담은 확실히 줄여주지만 DB를 자유롭게 변경해서 사용할 수 있는 유연한 코드를 보장해주지는 못한다. 현실적으로 DB를 자유롭게 바꾸어 사용할 수 있는 DB 프로그램을 작성하는 데는 두 가지 걸림돌이 있다.

- 비표준 SQL

DB의 변경 가능성을 고려해서 유연하게 만들어야 한다면 SQL은 제법 큰 걸림돌이 된다. 

→ 사용할 수 있는 방법은 DAO를 DB별로 만들어 사용하거나 SQL을 외부에서 독립시켜서 바꿔 쓸 수 있게 하는 것이다

- 호환성 없는 SQLException의 DB 에러정보

DB마다 SQL만 다른 것이 아니라 에러의 종류와 원인도 제각각이라는 점이 다. 그래서 JDBC는 데이터 처리 중에 발생하는 다양한 예외를 그냥 SQLException 하나에 모두 담아버린다. JDBC API는 이 SQLException 한 가지만 던지도록 설계되어 있다. 예외가 발생한 원인은 SQLException 안에 담긴 에러 코드와 SQL 상태정보를 참조해 봐야 한다. 그런데 SQLException 의 getErrorCode() 로 가져올 수 있는 DB 에러 코드는 DB별로 모두 다르다. DB 벤더가 정의한 고유한 에러 코드를 사용하기 때문이다.

→ SQLException 은 예외가 발생했을 때의 DB 상태를 담은 SQL 상태정보를 부가적으로 제공하지만, 문제는 DB의 JDBC 드라이버에서 SQLException 을 담을 상태 코드를 정확하게 만들어주지 않는다는 점이다.결국 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 SQLException 만으로 DB에 독립적인 유연한 코드를 작성하는 건 불가능에 가깝다.

2) DB에러코드 매핑을 통한 전략

SQLException의 비표준 에러 코드와 SQL 해결 방법은 DB별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석해 주는 기능을 만드는 것이다. 

→ 키 값이 중복돼서 중복 오류가 발생하는 경우에 MySQL 이라면 1062, 오라클이라면 1, DB2라면 -803이라는 에러 코드를 받게 된다. 이런 에러 코드 값을 확인할 수 있다면, 키 중복 때문에 발생하는 SQLException 을 DuplicateKeyException 이라는 의미가 분명히 드러나는 예외로 전환할 수 있다. DB 종류에 상관없이 동일한 상황에서 일관된 예외를 전달받을 수 있다면 효과적인 대응이 가능하다.

문제는 DB마다 에러 코드가 제각각이라는 점이다.

→ 스프링은 DB별 에러 코드를 분류해서 스프링이 정의한 예외 클래스와 매핑해놓은 에러 코드 매핑정보 테이블을 만들어두고 이를 이용한다.

```java
<bean id="Oracle" class="org.springframework.jdbc.support.SQLErrorCodes"> <property name="badSqlGrammarCodes"> <value>900,903,904,917,936,942,17006</value> </property> <property name="invalidResultSetAccessCodes"> <value>17003</value> </property> <property name="duplicateKeyCodes"> <value>1</value> </property> <property name="dataIntegrityViolationCodes"> <value>1400,1722,2291,2292</value> </property> <property name="dataAccessResourceFailureCodes"> <value>17002,17447</value> </property> ...
```

JdbcTemplate 은 SQLException 을 단지 런타임 예외인 DataAccessException 으로 포장하는 것이 아니라 DB의 에러 코드를 DataAccessException 계층구조클래스 중 하나로 매핑해준다. 전환되는 JdbcTemplate 에서 던지는 예외는 모두 DataAccessException 의 서브클래스 타입이다.

```java
public void add() throws DuplicateKeyException {  }
```

중복 키 에러를 따로 분류해서 예외처리를 해줬던 add() 메소드를 스프링의 JdbcTemplat e을 사용하도록 바꾸면 이와 같이 간단해진다.

JdbcTemplate 을 이용한다면 JDBC에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 된다. 그런데 중복키 에러가 발생했을 때 애플리케이션에서 직접 정의한 예외를 발생시키고 싶을 수 있다.

```java
public void add() throws DuplicateUserIdException { try {  } catch(DuplicateKeyException e) { throw new DuplicateUserIdException(e); } }
```

애플리케이션 레벨의 체크 예외인 DuplicateUserIdException 을 던지게 하고 싶다면 스프링의 DuplicateKeyException 예외를 전환해주는 코드를 DAO 안에 넣으면 된다.

하지만 SQLExecption 의 서브클래스이므로 여전히 체크 예외라는 점과 그 예외를 세분화하는 기준이 SQL 상태정보를 이용한다는 점에서 여전히 문제점이 있다.

3) DAO 인터페이스와 DataAccessException

스프링이 왜 DataAccessException 계층구조를 이용해 기술에 독립적인 예외를 정의하고 사용하게 하는지 생각해보자.

- DAO 인터페이스와 구현의 분리

DAO를 굳이 따로 만들어서 사용하는 이유는 데이터 액세스 로직을 담은 코드를 성격이 다른 코드에서 분리해놓기 위해서다. 또한 분리된 DAO 는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있게 만들기 위해서이기도 하다.

그런데 DAO의 사용 기술과 구현 코드는 전략 패턴과 DI를 통해서 DAO를 사용하는 클라이언트에게 감출 수 있지만, 메소드 선언에 나타나는 예외정보가 문제가 될 수 있다. UserDao 의 인터페이스를 분리해서 기술에 독립적인 인터페이스로 만들려면 아래와 같이 정의해야 한다.

```java
public interface UserDao { public void add(User user); ...
```}

하지만 이 메소드 선언은 사용할 수 없다. DAO에서 사용하는 데이터 액세스 기술의 API가 예외를 던지기 때문이다. 만약 JDBC API를 사용하는 UserDao 구현 클래스의 add() 메소드라면 SQLException 을 던질 것이다. 인터페이스의 메소드 선언에는 없는 예외를 구현 클래스 메소드의 throws 에 넣을 수는 없다. 따라서 인터페이스 메소 드도 다음과 같이 선언돼야 한다.

```java
public void add(User user) throws SQLException;
```

이렇게 정의한 인터페이스는 JDBC가 아닌 데이터 액세스 기술로 DAO 구현을 전환하면 사용할 수 없다. 데이터 액세스 기술의 API는 자신만의 독자적인 예외를 던지기 때문이다.

가장 단순한 해결 방법은 모든 예외를 다 받아주는 throws Exception 으로 선언하는 것이다.

```java
public void add(User user) throws Exception;
```

간단하긴 하지만 무책임한 선언이다. 다행히도 J D B C 보다는 늦게 등장한 J D O , H i b e r n a te , J PA 등의 기술은 SQLException 같은 체크 예외 대신 런타임 예외를 사용한다. 따라서 throws 에 선언을 해주지 않아도 된다. 

SQLException 을 던지는 JDBC API를 직접 사용하는 DAO는 메소드 내에서 런타임 예외로 포장해서 던져줄 수 있다. JDBC를 이용한 DAO에서 모든 SQLException 을 런타임 예외로 포장해주기만 한다면 DAO의 메소드는 처음 의도했던 대로 다음과 같이 선언해도 된다.

```java
public void add(User user);
```

이제 DAO에서 사용하는 기술에 완전히 독립적인 인터페이스 선언이 가능해졌다. 

대부분의 데이터 액세스 예외는 애플리케이션에서는 복구 불가능하거나 할 필요가 없는 것이다. 그렇다고 모든 예외를 다 무시해야 하는 건 아니다. 문제는 데이터 액세스 기술이 달라지면 같은 상황에서도 다른 종류의 예외가 던져진다는 점이다.

→ DAO를 사용하는 클라이언트 입장에서는 DAO의 사용 기술에 따라서 예외 처리 방법이 달라져야 한다. 결국 클라이언트가 DAO의 기술에 의존적이 될 수밖에 없다. 단지 인터페이스로 추상화하고, 일부 기술에서 발생하는 체크 예외를 런타임 예외로 전환하는 것만으론 불충분하다.

- 데이터 액세스 예외 추상화와 DataAccessException 계층구조

그래서 스프링은 자바의 다양한 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리해놓았다.

DataAccessException 은 자바의 주요 데이터 액세스 기술에서 발생할 수 있는 대부분의 예외를 추상화하고 있다. 데이터 액세스 기술에 상관없이 공통적인 예외도 있지만 일부 기술에서만 발생하는 예외도 있다. 스프링의 DataAccessException 은 이런 일부 기술에서만 공통적으로 나타나는 예외를 포함해서 데이터 액세스 기술에서 발생 가능한 대부분의 예외를 계층구조로 분류해놓았다.

또는 JDO, JPA, 하이버네이트처럼 오브젝트/엔티티 단위로 정보를 업데이트하는 경우에는 낙관적인 락킹 optimistic locking 이 발생할 수 있다. 이 낙관적인 락킹은 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 순차적으로 업데이트를 할 때, 뒤늦게 업데 이트한 것이 먼저 업데이트한 것을 덮어쓰지 않도록 막아주는 데 쓸 수 있는 편리한 기능이다. 이런 예외들은 사용자에게 적절한 안내 메시지를 보여주고, 다시 시도할 수 있도록 해줘야 한다. 하지만 역시 JDO, JPA, 하이버네이트마다 다른 종류의 낙관적인 락킹 예외를 발생시킨다. 그런데 스프링의 예외 전환 방법을 적용하면 기술에 상관없이 DataAccessException 의 서브클래스인 ObjectOptimisticLockingFailureException 으로 통일시킬 수 있다.


JdbcTempate 과 같이 스프링의 데이터 액세스 지원 기술을 이용해 DAO를 만들면 사용 기술에 독립적인 일관성 있는 예외를 던질 수 있다. 결국 인터페이스 사용, 런타임 예외 전환과 함께 DataAccessException 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO를 만들 수가 있다.

4) 기술에 독립적인 UserDao 만들기

지금까지 만들어서 써왔던 UserDao 클래스를 이제 인터페이스와 구현으로 분리해본다. 

- 인터페이스 적용

UserDao 인터페이스에는 기존 UserDao 클래스에서 DAO의 기능을 사용하려는 클라 이언트들이 필요한 것만 추출해내면 된다.

```java
public interface UserDao { void add(User user); User get(String id); List<User> getAll(); void deleteAll(); int getCount(); }
```

이제 기존의 UserDao 클래스는 다음과 같이 이름을 UserDaoJdbc 로 변경하고 UserDao 인터페이스를 구현하도록 implements 로 선언해줘야 한다.

```java
public class UserDaoJdbc implements UserDao {
```}

또 한 가지 변경할 사항은 스프링 설정파일의 userDao 빈 클래스 이름이다. userDao 빈 클래스를 다음과 같이 바꿔준다.

```java
<bean id="userDao" class="springbook.dao.UserDaoJdbc"> <property name="dataSource" ref="dataSource" /> </bean>
```

- 테스트 보완

UserDao 인스턴스 변수 선언도 UserDaoJdbc 로 변경해야 할까? 굳이 그럴 필요는 없다. 

중요한 건 테스 트의 관심이다. 그 구현 기술에 상관없이 DAO의 기능이 동작하는 데만 관심이 있다면, UserDao 인터페이스로 받아서 테스트하는 편이 낫다. 나중에 다른 데이터 액세스 기술로 DAO 빈을 변경한다고 하더라도 이 테스트는 여전히 유효하다.

 반면에 특정 기술을 사용한 UserDao 의 구현 내용에 관심을 가지고 테스트하려면 테스트에서 @Autowired 로 DI 받을 때 UserDaoJdbc 나 UserDaoHibernate 같이 특정 타입을 사용해야 한다. 일단 UserDao 테스트는 DAO의 기능을 검증하는 것이 목적이지 JDBC를 이용한 구현에 관심이 있는 게 아니다. 그러니 UserDao 라는 변수 타입을 그대로 두고 스프링 빈을 인터페이스로 가져오도록 만드는 편이 낫다. 

그림은 UserDao 의 인터페이스와 구현을 분리함으로써 데이터 액세스의 구체적인 기술과 UserDao 의 클라이언트 사이에 DI가 적용된 모습을 보여준다


- DataAccessException 활용 시 주의사항

이렇게 스프링을 활용하면 DB 종류나 데이터 액세스 기술에 상관없이 키 값이 중복이 되는 상황에서는 동일한 예외가 발생하리라고 기대할 것이다. 하지만 안타깝게도 DuplicateKeyException 은 아직까지는 JDBC를 이용하는 경우에만 발생한다. 데이터 액세스 기술을 하이버네이트나 JPA를 사용했을 때도 동일한 예외가 발생할 것으로 기대하 지만 실제로 다른 예외가 던져진다. 그 이유는 SQLException 에 담긴 DB의 에러 코드를 바로 해석하는 JDBC의 경우와 달리 JPA나 하이버네이트, JDO 등에서는 각 기술이 재정의한 예외를 가져와 스프링이 최종적으로 DataAccessException 으로 변환하는데, DB 의 에러 코드와 달리 이런 예외들은 세분화되어 있지 않기 때문이다.

DataAccessException 이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만 근본적인 한계 때문에 완벽하다고 기대할 수는 없다. 따라서 사용에 주의를 기울여야 한다. DataAccessException 을 잡아서 처리하는 코드를 만들려고 한다면 미리 학습 테스트를 만들어서 실제로 전환되는 예외의 종류를 확인해둘 필요가 있다.