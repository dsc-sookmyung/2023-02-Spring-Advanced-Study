# 3장 템플릿

## 🌿 다시 보는 초난감 DAO

아직 심각한 문제점이 나와있음: 예외 처리!

### 🌱 예외처리 기능을 갖춘 DAO

DB 커넥션은 제한적인 리소스임. <br>
이런 제한적인 리소스를 공유해 사용한는 서버에서는 JDBC 코드에 반드시 지켜야 할 원칙이 있음.

= **예외 처리!**

> 시스템의 심각한 문제를 막기 위해, <br>
> 정상적인 JDBC 코드의 흐름을 따르지 않고 중간에 어떤 이유로든 <br>
> 예외가 발생했을 경우에도 사용한 리소스를 반환하도록 만들어야 함.

#### JDBV 수정 기능의 예외처리 코드

```java
public void deleteAll() throws SQLException {
    Connection c = dataSource.getConnection();

    /**
     * 여기서 예외가 발생하면 바로 메소드 실행이 중단된다.
     *
    **/
    PreparedStatement ps = c.preparedStatement("delete from users");
    ps.executeUpdate();

    ps.close();
    c.close();
}
```

- 만약 위 코드에서 PreparedSTATEMENT를 처리하는 중에 예외가 발생한다면, 이 때는 메소드 실행을 끝마치지 못하고 바로 메소드를 빠져나가게 된다.

이 때 문제는, `close();` 메소드가 실행되지 않아서 **제대로 리소스가 반환되지 않을 수 있다는 점**이다!

일반적으로 서버에서는 제한된 개수의 DB 커넥션을 생성하여 재사용 가능한 풀로 관리한다.

- 이때는 명시적으로 `close()` 를 통해 커넥션을 돌려줘야만 다음 커넥션 요청에서 재사용 할 수 있다.
- 그러므로 이런 식으로 오류가 날 때마다 미처 반환되지 못한 Connection이 계속 증가하면 **어느 순간 커넥션 풀의 여유가 사라지고 리소스가 모자르게 될 것!**

<br>

따라서 이러한 JDBC 코드에서는 어떤 상황에서도 가져온 리소스를 반환하는 **try/catch/finally 구문**을 사용해야 한다.

```java
public void deleteAll() throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();
        ps = c.preparedStatement("delete from users");
        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null) {
            try {
                ps.close(); // 커넥션 반환
            } catch (SQLException e) {
            }
        }
        if (ps != null) {
            try {
                c.close(); // 커넥션 반환
            } catch (SQLExeption e) {
            }
        }
    }
}
```

예외가 어느 시점에 나는가에 따라 Connection과 PreparedStatement 중 어떤 것의 `close()` 를 호출해야 할 지가 달라진다.

- 만약 아직 커넥션을 가져오지 못한 상태에서 `close()`를 시도하면, `NullPointerException`이 발생됨.
- 그러므로 finally{} 안에서 null 체크를 통해 `close()` 시도가 가능한 상태인지 확인하고, 커넥션을 반환해준다.

#### JDBC 조회 기능의 예외 처리

- 조회를 위한 JDBC 코드는 더 복잡해진다. Connection, PreparedStatement, ResultSet 까지 필요하기 때문!

```java
public int getCount() throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;
    ResultSet rs = null;

    try {
        c = dataSource.getConnection();
        ps = c.preparedStatement("select * from users");

        rs = ps.executeQuery();
        rs.next();
        return rs.getInt(1);
    } catch (SQLException e) {
        throw e;
    } finally {

        /**
         * c -> ps -> rs 순서대로 커넥션을 생성했기 때문에,
         * 반대로 rs -> ps -> c 순서대로 커넥션을 반환한다.
        **/

        if (ps != null) {
            try {
                rs.close(); // 커넥션 반환
            } catch (SQLException e) {
            }
        }
        if (ps != null) {
            try {
                ps.close();
            } catch (SQLException e) {
            }
        }
        if (ps != null) {
            try {
                c.close();
            } catch (SQLExeption e) {
            }
        }
    }
}
```

- 여기서 finally{} 절 안에 있는 close()의 순서는 **만들어진 순서의 반대로 하는 것이 원칙**임.

## 🌿 변하는 것과 변하지 않는 것

### 🌱 JDBC try/catch/finally 코드의 문제점

- 너무 복잡한 try/catch/finally 블록이 2중으로 중첩되어 나옴.
- 모든 메소드마다 반복됨.

이렇게 반복되는 코드들은 COPY & PASTE 를 통해 반복해서 사용하면 매우 편하지만, <br>
실수를 했을 경우 치명적인 결과를 낳고, 실수를 찾아내기도 어렵다.

**변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자주 확장되고 자주 변하는 코드를 잘 분리해내야 한다!**

### 🌱 분리와 재사용을 위한 디자인 패턴 적용

```java
public void deleteAll() throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();

        // 이 줄만 변하는 부분임!
        ps = c.preparedStatement("delete from users");

        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null) {
            try {
                ps.close();
            } catch (SQLException e) {
            }
        }
        if (ps != null) {
            try {
                c.close();
            } catch (SQLExeption e) {
            }
        }
    }
}
```

우리가 작성한 `deleteAll()`의 코드를 보면 아래의 변하는 부분을 제외하면 전부 변하지 않는 부분임.

- `deleteAll()` 에서 변하는 부분

  ```java
    ps = c.preparedStatement("delete from users");
  ```

- `add()` 에서 변하는 부분

  ```java
    ps = c.preparedStatement("insert into users(id, name, password) values(?, ?, ?)");

    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());
  ```

> 로직에 따라 변하는 부분을 변하지 않는 나머지 코드에서 분리해서 <br>
> 변하지 않는 부분을 재사용할 수 있는 방법이 있을까?

#### 메소드 추출

변하는 부분을 메소드로 추출한다.

```java
public void deleteAll() throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();

        // 변하는 부분을 메소드로 추출한다.
        ps = makeStatement(c)

        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
      ...
    }
}

private PreparedStatement makeStatement(Connection c) throws SQLException {
    PreparedStatement ps;
    ps = c.preparedStatement("delete from users");
    return ps;
}
```

하지만 위 방법은 분리시키고 남은 메소드가 재사용이 필요한 부분이고, 분리된 메소드는 DAO 로직마다 새롭게 만들어서 학장돼야 하는 부분임.

재사용이 필요한 부분이 잘못 분리가 되었다.

#### 템플릿 메소드 패턴의 적용

**템플릿 메소드 패턴**? *상속*을 통해 기능을 확장해서 사용하는 부분임.

- 변하지 않는 부분: 슈퍼클래스에 위치시킴.
- 변하는 부분: 추상메소드로 정의하여 서브클래스에서 오버라이드하여 새롭게 정의해서 쓰도록 하는 것!

> UserDao를 추상 클래스로 변경하고, <br>
> 변하는 부분인 `makeStatment()` 메소드를 추상메소드로 정의한다.

```java
abstract protected PreparedStatement makeStatement(Connection c)
    throws SQLException;
```

<br>

이제 상속을 통해 클래스의 기능을 확장할 수 있으나, 여전히 제한은 남아있다.

1. 가장 큰 문제는 **DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 점**이다.
2. 또 확장구조가 이미 클래스를 설계하는 시점에서 고정되어 버린다.
   - 변하지 않는 코드를 가진 UserDao의 JDBC `try/catch/finally` 블록과 변하는 PreparedStatement를 담고 있는 서브 클래스들이 이미 클래스 레벨에서 컴파일 시점에 그 관계까 결정되어 있다.

#### 전략 패턴의 적용

전략 패턴! <br>
: 오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만드는 전략 패턴.

- OCP(개방 폐쇄 원칙)를 잘 지키는 구조
- 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어남.

<br>

OCP관점에서 봤을 때, 확장에 해당하는 변하는 부분을 **별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식**.

<img width="900" src="./img/week3/strategy-pattern_architecture.PNG">

- 좌측의 Context의 `contextMethod()`에서 일정한 구조를 가지고 동작하다가,
- **특정 확장 기능**은 Strategy 인터페이스를 통해 외부의 독립된 전략 클래스에 위임함.

<br>

`deleteAll()` 에 적용하면?

- 변하지 않는 부분 = contextMethod()

<br>

`deleteAll()`의 컨텍스트

1. DB 커넥션 가져오기
2. PreparedStatement를 만들어줄 외부 기능 호출하기
3. 전달받은 PreparedStatement 실행하기
4. 예외가 발생하면 이를 다시 메소드 밖으로 던지기
5. 모든 경우에 만들어진 PreparedStatement와 Connection을 적절히 닫아주기

여기서 두번째 작업인 **PreapredStatement를 만들어주는 외부 기능이 전략 패턴에서 말하는 전략**!

전략 패턴에 따라 이 기능을 인터페이스로 만들고 PreparedStatement 생성 전략을 호출하면 됨.

- PreparedStatement를 만드는 전략의 인터페이스는 컨텍스트가 만들어둔 Connection을 전달 받아서 PreparedStatement를 만들고, 만들어진 PreparedStatement 오브젝트를 돌려줌.

```java
public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c)
        throws SQLException;
}
```

위 인터페이스를 구현한 PreparedStatement를 생성하는 클래스를 만들어준다.

```java
public class DeleteAllStatement implements StatementStrategy {
    public PreapredStatement makePreparedStatement(Connection c)
    throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");

        return ps;
    }
}
```

이 `DeleteAllStatement` 클래스를 deleteAll() 메소드에서 사용하면?

```java
public void deleteAll() throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();

        StatementStrategy strategy = new DeleteAllStatement();
        ps = strategy.makePreparedStatement(c);

        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
      ...
    }
}
```

하지만 아직까지도 문제점은 남아있다.

전략 패턴은 필요에 따라 컨텍스트는 그대로 유지되면서 **전략을 바꿔쓸 수** 있어야 하는데, <br>
위 코드는 이미 구체적인 전략인 `DeleteAllStatement`를 사용하도록 고정되어 있다.

컨텍스트가 특정 구현 클래스인 `DeleteAllStatement`를 직접 알고 있다는 것이 전략 패턴에도 OCP에도 잘 들어맞는다고 볼 수 없다.

#### DI 적용을 위한 클라이언트/컨텍스트 분리

전략패턴의 실제적인 사용 방법을 살펴보자.

- Context가 어떤 전략을 사용하게 할 것인가는 앞단의 Client가 결정하는 게 일반적임.
- Client가 구체적인 전략의 하나를 선택하고 오브젝트로 만들어서 Context로 전달해주는 것.
- Context는 전달받은 그 Strategy 구현 클래스의 오브젝트를 사용함.

<img width="900" src="./img/week3/strategy-pattern_client.PNG">

<br>

해당 패턴에서 가장 중요한 것은 <br>
이 컨텍스트에 해당하는 **JDBC try/catch/finally 코드를 클라이언트 코드인 StatementStrategy를 만드는 부분에서 독립시켜야 한다는 것!**

아래 코드는 클라이언트에 들어가야하는 코드임.

```java
StatementStrategy strategy = new DeleteAllStatement();
```

클라이언트가 위 코드처럼 구체적인 전략을 설정하고, `jdbcContextWithStatementStrategy()` 메소드 파라미터로 던져주는 것!

- 따라서 클라이언트에서는 try/catch/finally 부분이 보이지 않음.

```java
public void jdbcContextWithStatementStrategy(StatementStrategy stmt)
    throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();

        ps = stmt.makePreparedStatement(c);

        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null) {
            try { ps.close(); } catch (SQLException e) {}
        }
        if (ps != null) {
            try { c.close(); } catch (SQLExeption e) {}
        }
    }
}
```

- 클라이언트로부터 StatementStrategy 타입의 전략 오브젝트를 제공받음
- JDBC try/catch/finally 구조로 만들어진 컨텍스트 내에서 작업을 수행함.
- 제공받은 StatementStrategy 오브젝트는 `PreparedStatment` 생성이 필요한 시점에 호출해서 사용한다!

<br>

클라이언트에 해당하는 부분은 다음과 같다.

```java
public void deleteAll() throws SQLException {
    // 전략 오브젝트 생성
    StatementStrategy st = new DeleteAllStatement();

    // 컨텍스트 호출. 전략 오브젝트 전달
    jdbcContextWithStatementStrategy(st);
}
```

- 전략 오브젝트를 만들고 컨텍스트를 호출하는 책임을 지님.

### 🌱 전체 코드 정리 및 정리

<details>
<summary> 🖥️ 전체 코드를 정리합니다! </summary>
<div markdown="6">

<br>

- 전체 아키텍처

<img width="900" src="./img/week3/strategy-pattern_client.PNG">

<br>

- 전략 인터페이스

```java
public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c)
        throws SQLException;
}
```

<br>

- 위의 전략 인터페이스를 구현한 클래스.
  - Delete 외에도 Read 등등의 전략을 StatementStrategy 인터페이스를 구현하여 생성할 수 있다.

```java
public class DeleteAllStatement implements StatementStrategy {
    public PreapredStatement makePreparedStatement(Connection c)
    throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");

        return ps;
    }
}
```

<br>

- 클라이언트 부분에서는 전략 오브젝트를 만들고 컨텍스트를 호출한다.
- 컨텍스트를 호출하는 `jdbcContextWithStatementStrategy` 메소드에서 전략에 맞는 실행과 try/catch/finally 예외 처리를 담당한다.

```java
public void deleteAll() throws SQLException {
    // 전략 오브젝트 생성
    StatementStrategy st = new DeleteAllStatement();

    // 컨텍스트 호출. 전략 오브젝트 전달
    jdbcContextWithStatementStrategy(st);
}

public void jdbcContextWithStatementStrategy(StatementStrategy stmt)
    throws SQLException {

    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();

        // 전략에 맞는 실행!
        ps = stmt.makePreparedStatement(c);

        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null) {
            try { ps.close(); } catch (SQLException e) {}
        }
        if (ps != null) {
            try { c.close(); } catch (SQLExeption e) {}
        }
    }
}
```

</div>
</details>

## 🌿 JDBC 전략 패턴의 최적화

deleteAll() 메소드에 담겨있던 변하는 부분과 변하지 않는 부분을 깔끔하게 분리해냄

독립된 작업 흐름이 담긴 jdbcContextithStatementStrategy()는 DAO 메소드들이 공유할 수 있게 됨.

DAO 메소드는 전략 패턴의 클라이언트로서 컨텍스트에 해당하는 jdbcContextWithStatmentStrategy() 메소드에 적절한 전략을 제공해주는 방법으로 사용할 수 있음.

여기서

- **컨텍스트 = PreparedStatement를 실행해주는 JDBC의 작업 흐름.**
- **전략 = PreparedStatement를 생성하는 것.**

### 🌱 전략 클래스의 추가 정보

이번엔 add() 메소드에도 적용해보자.

```java
public class AddStatement implements StatementStrategy {
    User user; // delete와 다르게 부가정보(user)가 필요함.

    // User는 생성자를 통해 주입받음.
    public AddStatement(User user) {
        this.user = user;
    }

    public PreparedStatement makePreparedStatement(Connection c)
        throws SQLException {

        PreparedStatement ps = c.prepareStatement(
            "insert into users(id, name, password) values(?,?,?)");

        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        return ps;
    }
}
```

<br>

따라서 이를 사용하는 클라이언트의 add() 메소드는 다음과 같이 작성할 수 있다.

```java
public void add(User user) throws SQLException {
    StatementStrategy st = new AddStatement(user);
    jdbcContextWithStatementStrategy(st);
}
```

### 🌱 전략과 클라이언트의 동거

아직 개선해야 할 점

- 새로운 DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 생성해야 한다는 점!
- DAO 메소드에서 StatementStrategy 에 전달할 User와 같은 부가정보가 있는 경우, 이를 위해 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 번거롭게 만들어야한다는 점.

#### 로컬 클래스

매번 StatementStrategy 전략 클래스를 독립된 파일로 만들지 말고 **UserDao 클래스 안의 내부 클래스**로 정의해버리자!

DeleteAllStatement나 AddStatement는 UserDao 밖에서는 사용되지 않음.

- UserDao의 메소드 로직에 강하게 결합되어있음.

```java
public void add(User user) throws SQLException {
    // add() 내부에 로컬 클래스로 AddStatement를 정의한다.
    class AddStatement implements StatementStrategy {
        User user;

        public AddStatement(User user) {
            this.user = user;
        }

        public PreparedStatement makePreparedStatement(Connection c)
            throws SQLException {

            PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");

            ps.setString(1, user.getId());
            ps.setString(2, user.getName());
            ps.setString(3, user.getPassword());

            return ps;
        }
    }

    StatementStrategy st = new AddStatement(user);
    jdbcContextWithStatementStrategy(st);
}
```

- 이렇게 정의된 로컬 클래스는 선언된 메소드 내에서만 사용이 가능하다.
  - 어차피 AddStatement class 가 사용될 곳이 add() 메소드 뿐이기에 이렇게 사용하기 전에 바로 정의해서 써준다!
- 클래스 파일도 줄고 코드 이해도 쉬워진다.

<br>

- 또한 내부 클래스는 메소드 내부에 정의되기 때문에 메소드 내부 변수들에 접근이 가능하다.
  - 메소드의 로컬 변수에 직접 접근이 가능하므로, <Br>
    여기서는 부가정보였던 User를 따로 생성자를 통해 주입을 받는 것이 아닌, **직접 접근해서 사용**할 수 있다.

```java
public void add(final User user) throws SQLException {
    class AddStatement implements StatementStrategy {
        public PreparedStatement makePreparedStatement(Connection c)
            throws SQLException {

            PreparedStatement ps = c.prepareStatement(
                "insert into users(id, name, password) values(?,?,?)");

            // 내부 클래스에서 외부 메소드의 로컬 변수에 접근 가능함.
            // 여기서는 user 변수에 접근하였음.
            ps.setString(1, user.getId());
            ps.setString(2, user.getName());
            ps.setString(3, user.getPassword());

            return ps;
        }
    }

    // 생성자 파라미터로 전달할 필요가 사라짐.
    StatementStrategy st = new AddStatement();
    jdbcContextWithStatementStrategy(st);
}
```

#### 익명 내부 클래스

여기서 더 업그레이드 한다면?

AddStatement 클래스는 add() 메소드에서만 사용할 용도로 만들어짐.

- 따라서 더 간결하게 클래스 이름도 제거할 수 있는 것!
- **익명 내부 클래스**를 사용해보자.

```java
public void add(final User user) throws SQLException {

    // 구현하는 인터페이스를 생성자처럼 이용한다.
    jdbcContextWithStatementStrategy(
        new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c)
                throws SQLException {

                PreparedStatement ps = c.prepareStatement(
                    "insert into users(id, name, password) values(?,?,?)");

                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getPassword());

                return ps;
            }
    });
}
```

- deleteAll() 메소드도 다음과 같이 변경할 수 있다.

```java
public void deleteAll() throws SQLException {

    // 구현하는 인터페이스를 생성자처럼 이용한다.
    jdbcContextWithStatementStrategy(
        new StatementStrategy() {
            public PreparedStatement makePreparedStatement(Connection c)
                throws SQLException {

                return c.prepareStatement("delete from users");
            }
    });
}
```

## 🌿 컨텍스트와 DI

### 🌱 JdbcContext의 분리

- UserDao의 메소드 = 클라이언트
- 익명 내부 클래스 = 개별 전략
- `jdbcContextWithStatementStrategy()` 메소드 = 컨텍스트
  - 컨텍스트 메소드는 UserDao 내의 PreparedStatement를 실행하는 기능을 가진 메소드에서 공유 가능.

`jdbcContextWithStatementStrategy()`는 UserDao외에도 다른 DAO에서도 사용가능해야 한다!

위 메소드를 UserDao 클래스 밖으로 독립시켜서 모든 DAO에서 사용 가능하게 설정해보자.

#### 클래스의 분리

- 분리해서 만들 클래스 = `JdbcContext`

```java
public class JdbcContext {
    private DataSource dataSource;

    public void setDataSource(DataSource datasource) {
        this.dataSource = dataSource;
    }

    public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        try {
            c = this.dataSource.getConnection();

            ps = stmt.makePreparedStatement(c);
            ps.executeUpdate();
        } catch (SQLException e) {
            throw e;
        } finally {
            if (ps != null) { try { ps.close(); } catch (SQLException e){} }
            if (c != null) { try { c.close(); } catch (SQLException e){} }
        }
    }
}

```

```java
public class UserDao {
    ...
    private JdbcContext jdbcContext;

    // JdbcContext를 DI 받도록 만듦.
    public void setJdbcContext(JdbcContext jdbcContext) {
        this.jdbcContext = jdbcContext;
    }

    public void add(final User user) throws SQLException {
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy() {...}
        );
    }

    public void deleteAll(final User user) throws SQLException {
        this.jdbcContext.workWithStatementStrategy(
            new StatementStrategy() {...}
        );
    }
}
```

### 🌱 JdbcContext의 특별한 DI

## 🌿 템플릿과 콜백

### 🌱 템플릿/콜백의 동작원리

### 🌱 편리한 콜백의 재활용

### 🌱 템플릿/콜백의 응용

## 🌿 스프링의 JDBC TEMPLATE

### 🌱 update()

### 🌱 queryForInt()

### 🌱 queryForObject()

### 🌱 query()

### 🌱 재사용 가능한 콜백의 분리
