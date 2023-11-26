![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/cbca6388-fee3-4c8a-851e-7cc04b312476)# 3장 템플릿

개방 폐쇄 원칙(OCP)은 확장에는 자유롭게 열려 있고 변경에는 굳게 닫혀 있다는 객체지향 설계의 핵심 원칙이다. 템플릿이란 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜서 효과적으로 활용할 수 있도록 하는 방법이다.

## 3.1 다시 보는 초난감 DAO

- 일반적으로 서버에서는 제한된 개수의 DB 커넥션을 만들어서 재사용 가능한 풀로 관리한다.
- DB 풀은 매번 getConnection()으로 가져간 커넥션을 명시적으로 close()해서 돌려줘야지만 다시 풀에 넣었다가 다음 커넥션 요청이 있을 때 재사용할 수 있다.
- 그런데 예외처리가 이뤄지지 못하고 미처 반환되지 못한 Connection이 계속 쌓이면 어느 순간 리소스가 모자란다는 심각한 오류와 함께 서버가 중단될 수 있다.

▶️ 예외 상황에서도 리소스를 제대로 반환할 수 있도록 try/catch/finally를 적용하자.

```java
public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;
    
    try {
        c = dataSource.getConnection();
        ps = c.prepareStatement("delete from users");
        ps.executeUpdate();     //  예외가 발생할 수 있는 코드를 모두 try 블록으로 묶어준다.
    } catch (SQLException e) {
        throw e;        //  예외가 발생했을 때 부가적인 작업을 해줄 수 있도록 catch 블록을 둔다. 아직은 예외를 메소드 밖으로 던지는 것 밖에 없다.
    } finally {         //  finally이므로 try 블록에서 예외가 발생했을 떄나 안 했을 때나 모두 실행된다.
        if (ps != null) {  //어느 시점에서 예외가 발생했는지에 따라서 close()를 사용할 수 있는 변수가 달라짐 -> null 확인 후 close();
            try {
                ps.close();
            } catch (SQLException e) {} //  ps.close() 메소드에서도 SQLException이 밣생할 수 있기 때문에 잡아줘야한다.
        }
        if (c != null) {
            try {
                c.close();
            } catch (SQLException e) {}
        }
    }
}
```

## 3.2 변하는 것과 변하지 않는 것

### JDBC try/catch/finally 코드의 문제점

- 복잡한 try/catch/finally 블록이 2중으로 중첩까지 되어 나오는데다, 모든 메소드마다 반복된다.

- 복붙 → 서비스 중단 → 세계 멸망 💥

- 이 문제의 핵심은 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리해내는 작업이다.

### 분리와 재사용을 위한 디자인 패턴 적용

- 메소드 추출 → 🚫
    
    ```java
    public void deleteAll() throws SQLException {
        ...
        try {
            c = dataSource.getConnectin();
            ps = makeStatement(c);      //  변하는 부분을 메소드로 추출하고 변하지 않는 부분에서 호출한다.
            ps.executeUpdate();
        } catch (SQLException e) {...}
    }
    
    private PreparedStatement makeStatement(Connection c) throws SQLException {
        PreparedStatement ps;
        ps = c.preparedStatement("delete from users");
        return ps;
    }
    ```
    
    메소드 추출 리팩토링을 적용하는 경우에는 분리시킨 메소드를 다른 곳에서 재사용할 수 있어야 하는데, 이건 반대로 분리시키고 남은 메소드가 재사용이 필요한 부분이고, 분리된 메소드는 메소드는 DAO 로직마다 새롭게 만들어서 확장돼야하는 부분이다.
    
- 템플릿 메소드 패턴의 적용
    
    > 💡 **템플릿 메소드 패턴**<br/>
    상속을 통해 기능을 확장해서 사용한다. 변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메소드로 정의해서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 한다.
    > 
    - 하지만 템플릿 메소드 패턴으로의 접근은 제한이 많다. DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다. 또 확장구조가 이미 클래스를 설계하는 시점에서 고정된다. 상속을 통해 확장을 꾀하는 템플릿 메소드 패턴의 단점이 그대로 드러난다.
- 전략 패턴의 적용
    
    ![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/f69028ca-954c-4375-9b79-497b757ab815)

    
    - 좌측에 있는 Context의 contextMethod()에서 일정한 구조를 가지고 동작하다가 특정 확장 기능은 Strategy 인터페이스를 통해 외부의 독립된 전략 클래스에 위임한다.
    
    ```java
    public void deleteAll() throws SQLException {
        ...
        try {
            c = dataSource.getConnection();
    
            StatementStrategy strategy = new DeleteAllStatement();  //  전략 클래스가 DeleteAllStatement로 고정됨으로써 OCP 개방 원칙에 맞지 않게 된다.
            ps = starategy.makePreparedStatement(c);
    
            ps.executeUpdate();
        } catch (SQLException e) {...}
    }
    ```
    
- DI 적용을 위한 클라이언트 컨텍스트 분리
    
    ![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/a12640c4-683c-49c0-864a-de0f24a7a02e)

    
    - 중요한 것은 이 컨텍스트에 해당하는 JDBC try/catch/finally 코드를 클라이언트 코드인 StatementStrategy를 만드는 부분에서 독립시켜야 한다는 점이다.
    
    ```java
    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;
    
        try {
            c = dataSource.getConnection();
            ps = stmt.makePreparedStatement(c);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw e;
        } finally {
            if (ps != null) { try { ps.close(); } catch (SQLException e) {}
            if (c != null) { try { c.close(); } catch (SQLException e) {}
        }
    }
    ```
    
    이 메소드는 클라이언트로부터 StatementStrategy 타입의 전략 오브젝트를 제공받고 JDBC try/catch/finally 구조로 만들어진 컨텍스트 내에서 작업을 수행한다.
    

## 3.3 JDBC 전략 패턴의 최적화

### 전략과 클라이언트의 동거

- 개선할 점
    - DAO 메소드마다 새로은 StatementStrategy 구현 클래스를 만들어야 한다. (런타임 시 DI가 다이내믹하게 일어난다는 점만 빼면 템플릿 메소드 패턴보다 나을 것이 없다.)
    - DAO 메소드에서 StatementStrategy에 전달한 User와 같은 부가적인 정보가 있는 경우, 이를 위해 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 번거롭게 만들어야 한다.
- 로컬 클래스
    - 클래스 파일이 많아지는 문제는 내부 클래스로 해결할 수 있다.
    - 생성 로직을 함께 볼 수 있어 코드 이해에 도움이 된다.
    - 로컬 클래스는 자신이 선언된 곳의 정보에 접근할 수 있다. → 번거롭게 생성자를 통해 User 오브젝트를 전달해줄 필요가 없다.
    
    ```java
    public void add(final User user) throws SQLException {
      class AddStatement implements StatementStrategy {   //  add() 메소드 내부에 선언된 로컬 클래
          User user;
    
          public AddStatement(User user) {
              this.user = user;
          }
    
          public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
              PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
              ...
          }
    
          StatementStrategy st = new AddStatement(user);
          jdbcContextWithStatementStrategy(st);
      }
    }
    ```
    
- 익명 내부 클래스
    
    ```java
    public void add(final User user) throws SQLException {
      class AddStatement implements StatementStrategy {   //  add() 메소드 내부에 선언된 로컬 클래
          User user;
    
          public AddStatement(User user) {
              this.user = user;
          }
    
          public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
              PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
              ...
          }
    
          StatementStrategy st = new AddStatement(user);
          jdbcContextWithStatementStrategy(st);
      }
    }
    ```
    

## 3.4 컨텍스트와 DI

### JdbcContext의 분리

jdbcContextWithStetementStrategy()는 다른 DAO에서도 사용 가능하다. UserDao 클래스 밖으로 독립시키자.

![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/6755d153-6371-4c36-87a9-3543125aaf5f)

### JdbcContext의 특별한 DI

UserDao와 JdbcContext 사이에는 인터페이스를 사용하지 않고 DI를 적용했다. 런타임 시에 DI 방식으로 외부에서 오브젝트를 주입해주는 방식을 사용하긴 했지만, 의존 오브젝트의 구현 클래스를 변경할 수는 없다.

- 스프링 빈으로 DI
    - 인터페이스를 사용하지 않았다면 엄밀히 온전한 DI라고는 볼 수 없다. 그러나 스프링의 DI는 넓게 보자면 객체의 생성과 관계설정에 대한 제어권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC 개념을 포괄한다.
    - JdbcContext를 UserDao와 DI구조로 만들어야 할 이유?
        
        1️⃣ JdbcContext가 스프링 컨테이너의 싱글톤 빈이 된다. 일종의 서비스 오브젝트로서 의미가 있고, 싱글톤으로 등록되어 여러 오브젝트에 공유해 사용되는 것이 이상적이다.
        
        2️⃣ Jdbc가 DI를 통해 다른 빈에 의존하고 있기 때문이다. DI를 위해서 주입되는/주입받는 양쪽 오브젝트가 스프링 빈으로 등록되어야 한다.
        
    - 왜 인터페이스를 사용하지 않았을까?
        
        : 이 둘은 강한 응집도를 갖고 있다. 이런 경우는 굳이 인터페이스를 두지 말고 강한 결합 관계를 허용하면서 스프링의 빈으로 등록해 DI가 되도록 만들어도 좋다.
        
- 코드를 이용하는 수동 DI
    
    <img width="323" alt="image" src="https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/be7338c8-6f79-4f61-997b-971dd4bea636">

    
    - JdbcContext의 생성과 초기화 → UserDao
    - DataSource 빈의 의존 → UserDao가 주입받아 제공
    - 인터페이스를 사용하지 않는 DI 방법 2가지 비교
        - 스프링의 DI 이용: 의존관계가 설정파일에 명확하게 드러난다. / DI의 근본원칙에 부합하지 않는 구체적인 클래스의 관계가 설정에 직접 노출
        - 수동 DI: 필요에 따라 내부에서 은밀히 DI 수행, 전략을 외부에 감출 수 있다. / 싱글톤으로 만들 수 없고, DI 작업을 위한 부가적인 코드가 필요하다.
        

## 3.5 템플릿과 콜백

> 💡 **템플릿 (template)**
> 
> 
> 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀을 가리킨다. 프로그래밍에서는 고정된 틀 안에 바꿀 수 있는 부분을 넣어서 사용하는 경우에 템플릿이라고 부른다. 템플릿 메소드 패턴은 고정된 틀의 로직을 가진 템플릿 메소드를 슈퍼클래스에 두고, 바뀌는 부분을 서브클래스 메소드에 두는 구조로 이뤄진다.
> 

> 💡 **콜백 (callback)**<br/>
 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 말한다. 파라미터로 전달되지만 값을 참조하기 위한 것이 아니라 특정 로직을 담은 메소드를 실행시키기 위해 사용한다. 자바에선 메소드 자체를 파라미터로 전달할 방법은 없기 때문에 메소가 담긴 오브젝트를 전달해야 한다. 그래서 펑셔널 오브젝트(functional object)라고도 한다.
> 

### 템플릿/콜백의 동작원리

템플릿/콜백 패턴의 콜백은 보통 단일 메소드 인터페이스를 사용한다.

![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/a9a43846-d8de-42b5-968a-fbd1c491876a)


- 클라이언트의 역할은 템플릿 안에서 실행될 로직을 담은 콜백 오브젝트를 만들고, 콜백이 참조할 정보를 제공하는 것이다. 만들어진 콜백은 클라이언트가 템플릿의 메소드를 호출할 때 파라미터로 전달된다.
- 템플릿은 정해진 작업 흐름을 따라 작업을 진행하다가 내부에서 생성한 참조정보를 가지고 콜백 오브젝트의 메소드를 호출한다. 콜백은 클라이언트 메소드에 있는 정보와 템플릿이 제공한 참조정보를 이용해서 작업을 수행하고 그 결과를 다시 템플릿에 돌려준다,
- 템플릿은 콜백이 돌려준 정보를 사용해서 작업을 마저 수행한다. 경우에 따라 최종 결과를 클라이언트에 다시 돌려주기도 한다.

### 편리한 콜백의 재활용

- 콜백의 분리와 재활용
    
    ```java
    public void deleteAll() throws SQLException {
        executeSql("delete from users");
    }
    
    private void executeSql(final String query) throws SQLException {
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy() {   //  변하지 않는 콜백 클래스 정의와 오브젝트 생성
                public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
                    return c.preparedStatement(query);
                }
            }
        );
    }
    ```
    
    바뀌지 않는 모든 부분을 빼내서 executeSql() 메소드로 만들었다. 이렇게 해서 재활용 가능한 콜백을 담은 메소드가 만들어졌다.
    
- 콜백과 템플릿의 결합
    
    ```java
    public class JdbcContext {
        ...
        public void executeSql(final String query) throws SQLException {
            workWithStatementStrategy(
                new StatementStrategy() {...}
            );
        }
    }
    ```
    
    UserDao만 사용하기에는 아까운 executeSql을 JdbcContext 안으로 옮기자.
    
    ![image](https://github.com/lizuAg/2023-02-Spring-Advanced-Study/assets/68546023/961c6300-14c9-45d9-a884-2ddfc3380f42)

    
    - 결국 JdbcContext 안에 클라이언트와 템플릿, 콜백이 모두 함께 공존하여 동작하는 구조가 되었다.
    - 일반적으로 성격이 다른 코드들은 가능한 한 분리하는 것이 낫지만, 이 경우는 하나의 목적을 위해 서로 긴밀하게 연관되어 동작하는 응집력이 강한 코드들이기 때문에 한 군데 모여있는 것이 유리하다.
    

## 3.7 정리

1) JDBC와 같은 예외가 발생할 가능성이 있으며 공유 리소스의 반환이 필요한 코드는 반드시 try/catch/finally 블록으로 관리해야한다.

2) 일정한 작업 흐름이 반복되면서 그중 일부 기능만 바뀌는 코드가 존재한다면 전략 패턴을 적용한다. 바뀌지 않는 부분은 컨텍스트로, 바뀌는 부분은 전략으로 만들고 인터페이스를 통해 유연하게 전략을 변경할 수 있도록 구성한다.

3) 같은 애플리케이션 안에서 여러 가지 종류의 전략을 다이내믹하게 구성하고 사용해야 한다면 컨텍스트를 이용하는 클라이언트 메소드에서 직접 전략을 정의하고 제공하게 만든다.

4) 클라이언트 메소드 안에 익명 내부 클래스를 사용해서 전략 오브젝트를 구현하면 코드도 간결해지고 메소드의 정보를 직접 사용할 수 있어서 편리하다.

5) 컨텍스트가 하나 이상의 클라이언트 오브젝트에서 사용된다면 클래스를 분리해서 공유하도록 만든다.

6) 컨텍스트는 별도의 빈으로 등록해서 DI받거나 클라이언트 클래스에서 직접 생성해서 사용한다. 클래스 내부에서 컨텍스트를 사용할 때 컨텍스트가 의존하는 외부의 오브젝트가 있다면 코드를 이용해서 직접 DI해줄 수 있다.

7) 단일 전략 메소드를 갖는 전략 패턴이면서 익명 내부 클래스를사용해서 매번 전략을 새로 만들어 사용하고, 컨텍스트 호출과 동시에 전략 DI를 수행하는 방식을 템플릿/콜백 패턴이라고 한다.

8) 콜백의 코드에도 일정한 패턴이 반복된다면 콜백을 템플릿에 넣고 재활용하는 것이 편리하다.

9) 템플릿과 콜백의 타입이 다양하게 바뀔 수 있다면 제네릭스를 이용한다.

10) 스프링은 JDBC 코드 작성을 위해 JdbcTemplate을 기반으로 하는 다양한 템플릿과 콜백을 제공한다.

11) 템플릿은 한 번에 하나 이상의 콜백을 사용할 수도 있고, 하나의 콜백을 여러 번 호출할 수도 있다.

12) 템플릿/콜백을 설계할 때는 템플릿과 콜백 사이에 주고받는 정보에 관심을 둬야 한다.
