# 4장 예외

## 사라진 SQLException

### 초난감 예외처리

- 예외를 잡고는 아무것도 하지 않는다.
    
    ```yaml
    try {
    	...
    } catch(SQLEXception e) {
    }
    
    ```
    
    예외가 발생하면 그것을 catch 블록을 써서 잡아내는 것까지는 좋은데 그리고 아무것도 하지 않고 별문제 없는 것처럼 넘어가 버리는 건 정말 위험한 일이다. 그것은 예외가 발생했는데 그것을 무시하고 계속 진행해버리기 때문이다.
    
- 예외가 발생하면 화면에 출력만 하고 끝낸다.
    
    ```yaml
    try {
    	...
    } catch(SQLEXception e) {
    	System.out.println(e);
    }
    
    try {
    	...
    } catch(SQLEXception e) {
    	e.printStackTrace();
    }
    ```
    
    콘솔 로그를 누군가가 계속 모니터링하지 않는 한 이 예외 코드는 심각한 폭탄으로 남아 있을 것이다. 모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 한다.
    

> ☘️ **예외를 처리할 때 반드시 지켜야할 핵심 원칙 한 가지** 
모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 한다.
> 
- 무의미하고 무책임한 throws
    
    ```yaml
    public void method1() throws Exception {
        method2();
    )
    public void method2() throws Exception {
        method3();
    }
    public void method3() throws Exception {
        ...
    }
    ```
    
    자신이 사용하려는 메소드에 throws Exception이 선언되어 있다면 정말 무엇인가 실행 중에 예외적인 상황이 발생할 수 있다는 것인지, 아니면 그냥 습관적으로 복사해서 붙여놓은 것인지 알 수가 없다. 결국 이런 메소드를 사용하는 메소드에서도 역시 throws Exception을 따라서 붙이는 수밖에 없다.
    

### 예외의 종류와 특징

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f99fbc47-4105-4bee-84c9-8fff10da1b47/b59c417a-fdb8-406e-9d25-6822b58d8c5c/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/f99fbc47-4105-4bee-84c9-8fff10da1b47/e9785311-c038-471b-958f-37db1039e94a/Untitled.png)

- Error
    - java.lang.Error 클래스의 서브클래스
    - 에러는 시스템에 뭔가 비정상적인 상황이 발생했을 경우에 사용된다. (OutOfMemoryError, ThreadDeath)
    - 시스템 레벨에서 특정한 작업을 하는 게 아니라면 애플리케이션에서 에러 처리는 신경 쓰지 않아도 된다.
- Exception (체크 예외 + 언체크 예외)
    - java.lang.Exception 클래스와 그 서브 클래스로 정의되는 예외
    - 개발자들이 만든 애플리케이션 코드의 작업 중에 예외 상황이 발생했을 경우에 사용된다.
- 체크 예외
    - Exception 클래스의 서브 클래스이면서 RuntimeException을 상속하지 않은 것.
    - 일반적으로 말하는 예외
    - 사용할 메소드가 체크 예외를 던진다면 이를 catch 문으로 잡든지, 아니면 다시 throws를 정의해서 메소드 밖으로 던져야 한다. 그렇지 않으면 컴파일 에러가 발생한다.
- RuntimeException과 언체크/런타임 예외
    - Exception 클래스의 서브 클래스이면서 RuntimeException을 상속한 것
    - 명시적인 예외처리를 강제하지 않는다. (개발자 부주의로 인함)
    - 런타임 예외는 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들이다.(NullPointerException, IllegalArgumentException)

### 예외처리 방법

- 예외 복구
    
    : 예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려 놓는다.
    
    - 예외처리 코드를 강제하는 체크 예외들은 예외를 어떤 식으로든 복구할 가능성이 있는 경우에 사용한다.
    
    ex) 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없다거나 다른 문제가 있어서 읽히지가 않아서 IOException이 발생 > 이때는 사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상황을 해결할 수 있다.
    
    ```yaml
    int maxretry = MAX_RETRY; 
    while(maxretry — > 0) {
        try {
            ... // 예외가 발생할 가능성이 있는 시도
            return;
        }
        catch(SomeException e) {
            // 작업 성공
            // 로그 출력. 정해진 시간만큼 대기
        } finally {
            // 리소스 반납. 정리 작업
        } 
    }
    throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
    ```
    
- 예외처리 회피
    
    : 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것이다.
    
    - throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것이다.
    
    ```yaml
    public void add() throws SQLException { 
      try {
        // JDBC API
      }
      catch(SQLException e) {
        // 로그 출력
    	  throw e;
    	} 
    }
    ```
    
    - DAO가 SQLException을 생각 없이 던져버리면 어떻게 될까? DAO에서 던진 SQLException을 서비스 계층 메소드가 다시 던지고, 컨트롤러도 다시 지도록 선언해서 예외는 그냥 서버로 전달되고 말 것이다. 예외를 회피하는 것은 예외를 복구하는 것처럼 의도가 분명해야 한다.
- 예외 전환
    
    : 예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것이다. 하지만 예외 회피와는 달리, 발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다는 특징이 있다.
    
    - 예외 전환은 보통 두가지 목적으로 사용된다.
        
        1) 내부에서 발생한 예외를 그대로 던지는 것이 그 예외 상황에 대한 적절한 의미를 부여해주지 못하는 경우에, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서
        
        ```yaml
        public void add(User user) throws DuplicateUserldException, SQLException { 
            try {
                // ]DBC를 이용해 user 정보를 애에 추가하는 코드 또는
                // 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드 
            }
            catch(SQLException e) {
                // ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환 
                if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
                    throw DuplicateUserIdException(); 
                else
                    throw e; // 그 외의 경우는 SQLException 그대로
            } 
        }
        ```
        
        보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외로 만드는 것이 좋다.
        
        ```yaml
        throw DuplicateUserldException(e);
        or
        throw DuplicateUserIdException().initCause(e);
        ```
        
        2) 두 번째 전환 방법은 예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 것이다. 중첩예외를 이용해 새로운 예외를 만들고 원인이 되는 예외를 내부에 담아서 던지는 방식은 같지만, 예외처리를 강제하는 체크 예외를 런타임 예외로 바꾸는 경우에 사용한다.
        

### 예외처리 전략

- 런타임 예외의 보편화
    - 일반적으로는 체크 예외가 일반적인 예외를 다루고, 언체크 예외는 시스템 장애나 프로그램상의 오류에 사용한다고 했다. 말 그대로 예외적인 상황이기 때문에 자바는 이를 처리하는 catch 블록이나 throws 선언을 강제하고 있다는 점이다.
    - DuplicatedUserIdException은 충분히 복구 가능한 예외이므로 add() 메소드를 사용하는 쪽에서 잡아서 대응할 수 있다. 하지만 SQLException은 대부분 복구 불가능한 예외이므로 잡아봤자 처리할 것도 없고, 결국 throws를 타고 계속 앞으로 전달되다가 애플리케이션 밖으로 던져질 것이다. 그럴 바에는 그냥 런타임 예외로 포장해 던져버려서 그 밖의 메소드들이 신경 쓰지 않게 해주는 편이 낫다.
    
    ```yaml
    public class DuplicateUserldException extends RuntimeException { 
        public DuplicateUserIdException(Throwable cause) {
            super(cause); 
        }
    }
    ```
    
    ```yaml
    public void add() throws DuplicateUserldException { 
        try {
            // ]DBC를 이용해 user 정보를 애에 추가하는 코드 또는
            // 그런 기능이 있는 다른 SQLException을 던지는 메소드를 호출하는 코드
        }
        catch (SQLException e) {
            if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
                throw new DuplicateUserldException(e); // 예외 전환: 처리할 수 있다면 런타임 예외로 만든다.
            else
                throw new RuntimeException(e); // 에외 포장: 복구 불가능한 예외이므로 런타임 예외로 포장.
        }
    }
    ```
    
- 애플리케이션 예외
    
    : 시스템 또는 외부의 상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch해서 조치를 취하도록 요구하는 예외
    
    ex) 출금 시 잔액 부족 상황
    

## 4.2 예외 전환

- 예외 전환의 목적은 두 가지라고 설명했다. 하나는 앞에서 적용해본 것처럼 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄여주는 것이고, 다른 하나는 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꿔서 던져주는 것이다.
- 스프링은 자바의 다양한 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 DataAccessException 계층 구조 안에 정리해두었다.
- 낙관적인 락킹(optimistic locking) 상황에서, 사용자들에게 적절한 안내와 다시 시도할 수 있도록 하기 위해서는 JDO, JPA, 하이버네이트 ORM 기술 각각의 예외를 처리해야한다.
- 이때는 슈퍼크래스를 상속해서 사용한다면 일관된 방식으로 처리할 수 있을 것이다.

## 4.3 정리

1) 예외를 잡아서 아무런 조취를 취하지 않거나 의미없는 throws 선언을 남발하는 것은 위험하다.

2) 예외는 복구하거나 예외처리 오브젝트로 의도적으로 전달하거나 적절한 예외로 전환해야 한다.

3) 좀 더 의미 있는 예외로 변경하거나, 불필요한 catch/throws를 피하기 위해 런타임 예외로 포장하는 두 가지 방법의 예외 전환이 있다.

4) 복구할 수 없는 예외는 가능한 한 빨리 런타임 예외로 전환하는 것이 바람직하다.

5) JDBC의 SQLException은 대부분 복구할 수 없는 예외이므로 런타임 예외로 포장해야 한다.

6) 스프링은 DataAccessException을 통해 DB에 독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공한다.

7) DAO를 데이터 액세스 기술에서 독립시키려면 인터페이스 도입과 예외 전환, 기술에 독립적인 추상화된 예외로 전환이 필요하다.
