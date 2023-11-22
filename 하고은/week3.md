- OCP(개방 폐쇄 원칙): 확장에는 열려있고, 변경에는 굳게 닫혀 있다.
  - 코드에서 기능이 다양해지고 확장하려는 성질을 가진 부분도 있고, 고정되고 변하지 않으려는 성질을 가진 곳도 있음
  - 독립적으로 변경될 수 있는 효율적인 구조를 만들어 줌

# 1️⃣ 초난감 DAO 리뷰

## 예외처리 기능을 갖춘 DAO

- JDBC 코드에 예외처리 필요

```java
public void deleteAll() throws SQLException {
    Connection c = dataSource.getConnection();

		PreparedStatement ps = c.prepareStatement("delete from users");
    ps.executeUpdate();

    ps.close();
    c.close();
}
```

- PreparedStatement 처리 중 오류가 발생할 경우 close()가 실행되지 않아 리소스가 제대로 반환되지 않을 수 있음
- DB풀은 getConnection()으로 가져간 후 명시적으로 close()해서 리소스를 반환해주어야 함
- Connection과 PreparedStatement는 정해진 풀 안에 제한된 후의 리소스를 만들어두고 필요할 때 할당한 후 반환하면 다시 풀에 넣는 방식으로 운영됨 (리소스가 고갈되는 문제가 발생할 수 있음)
- 반환되지 못한 Connection이 쌓이면서 심각한 오류로 서버가 중단될 수 있음

```java
public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();
        ps = c.prepareStatement("DELETE FROM users");
        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null)
            try {
                ps.close();
            } catch (SQLException e) {
                // Connection close()를 하지 못하고 메소드를 빠져나갈 수 있음
            }
        if (c != null)
            try {
                c.close();
            } catch (SQLException e) {
            }
        // ResultSet을 사용하는 메소드는 resultSet도 같은 방식으로 처리해주어야 함 (ex. getCount)
    }
}
```

### JDBC 조회 기능의 예외처리

- Connection, PreparedStatement + ResultSet 추가

```java
public class UserDao {
    // ...
    public int getCount() throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;
        ResultSet rs = null;

        try {
            c = dataSource.getConnection();
            ps = c.prepareStatement("select count(*) from users");
            rs = ps.executeQuery();
            rs.next();
            int count = rs.getInt(1);
            return count;
        } catch (SQLException e) {
            throw e;
        } finally {
            if (rs != null) {
                try {
                    rs.close();
                } catch (SQLException e) {

                }
            }

            if (ps != null) {
                try {
                    ps.close();
                } catch (SQLException e) {

                }
            }

            if (c != null) {
                try {
                    c.close();
                } catch (SQLException e) {

                }
            }
        }
    }
}
```

# 2️⃣ 변하는 것과 변하지 않는 것

- JDBC try/catch/finally 코드의 문제점: try/catch/finally 블록의 2중 중첩

## 분리와 재사용을 위한 디자인 패턴 적용

- 고정되는 부분(메소드에서 동일하게 나타날 수 있는 부분)과 로직에 따라 변하는 부분을 구분

### 메소드 추출

- 변하는 부분을 메소드로 빼기 → 좋은 점이 없음..

```java
private PreparedStatement makeStatement(Connection c) throws SQLException{
    PreparedStatement ps;
    ps = c.prepareStatement("delete from user");
    return ps;
}
```

### 템플릿 메소드 패턴의 적용

- 템플릿 메소드 패턴: 상속을 통해 기능을 확장해서 사용하는 부분 (변하지 않는 부분은 슈퍼 클래스, 변하는 부분은 추상 메소드로 정의 후 서브클래스에서 오버라이드)
- 추출해서 별도의 메소드로 독립시킨 makeStatement() 메소드를 추상 메소드 선언으로 변경
- UserDao 클래스도 추상 클래스로 변경

```java
public abstract PreparedStatement makeStatement(Connection c) throws SQLException;
```

- 상속하는 서브클래스를 만들어서 해당 메소드 구현

```java
public class UserDaoDeleteAll extends UserDao{
    private PreparedStatement makeStatement(Connection c) throws SQLException{
        PreparedStatement ps;
        ps = c.prepareStatement("delete from user");
        return ps;
    }
}
```

- 템플릿 메소드 패턴으로의 접근은 제한이 많음
  - DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 함
  - 확장구조가 클래스를 설계하는 시점에서 고정됨: 변하지 않는 JDBC의 try/catch/finally 블록과 변하는 PreparedStatement를 담고 있는 서브클래스들이 클래스 레벨에서 컴파일 시점에 관계가 결정됨 → 유연성이 떨어짐

### 전략 패턴의 적용

- 오브젝트를 둘로 분리하고 클래스 레벨에서 인터페이스를 통해서만 의존하게 하는 것
- 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임
- contextMethod()에서 일정한 구조를 가지고 동작 후 특정 확장 기능은 Strategy 인터페이스를 통해 외부의 독립된 전략 클래스에 위임

- contextMethod(): deleteAll() 메소드에서 변하지 않는 부분이라고 명시한 것
- JDBC를 이용해 DB를 업데이트하는 작업이라는 변하지 않는 맥락(context)을 가짐
  - DB 커넥션 가져오기
  - **_PreparedStatement를 만들어줄 외부 기능 호출 (전략 패턴의 전략)_**
    → 해당 컨텍스트 내에서 만들어둔 DB 커넥션 전달 필요
    → 인터페이스는 컨텍스트가 만들어둔 Connection을 전달받아서, PreparedStatement를 만든 후 만들어진 해당 오브젝트를 돌려줌
  - 전달받은 PreparedStatement 실행
  - 예외 발생 시 예외를 메소드 밖으로 던짐
  - 모든 경우에 만들어진 PreparedStatement와 Connection을 닫아주기

```java
public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```

```java
public class DeleteAllStatement implements StatementStrategy {
    @Override
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = c.prepareStatement("delete from users");
        return ps;
    }
}
```

```java
public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;

    try {
      c = dataSource.getConnection();

      StatementStrategy strategy = new DeleteAllStatement(); // 구체적인 전략 클래스 명시
      ps = strategy.makePreparedStatement(c);

      ps.executeUpdate();
    } catch (SQLException e) {
            ...
    }
```

- 🚨 구체적인 전략 클래스 명시 → 전략 패턴, OCP에 들어맞지 않음

### DI 적용을 위한 클라이언트/컨텍스트 분리

- Context가 어떤 전략을 사용하게 할 것인가

  → Context를 사용하는 앞단의 Client가 결정하는 게 일반적

  → Client가 구체적인 전략의 하나를 선택하고 오브젝트로 만들어서 Context에 전달

  → Context는 전달받은 Strategy 구현 클래스의 오브젝트 사용

- DI: 전략 패턴의 장점을 일반적으로 활용할 수 있도록 만든 구조

```java
StatementStrategy strategy = new DeleteAllStatement(); // 클라이언트에 들어가야 할 코드
```

- deleteAll()의 나머지 코드는 컨텍스트 코드이므로 분리 필요 (별도의 메소드로 독립)
- 클라이언트는 전략 클래스의 오브젝트(ex. DeleteAllStatement 오브젝트)를 컨텍스트 메소드로 전달해야 함 → StatementStrategy를 컨텍스트 메소드 파라미터로 지정 필요

```java
// 제공받은 전략 오브젝트는 PreparedStatement 생성이 필요한 시점에 호출해서 사용
public void jdbcContextWithStratementStrategy(StatementStrategy stmt) throws SQLException {
		Connection c = null;
		*PreparedStatement* ps = null;

		try {
				c = dataSource.getConnection();
		    ps = stmt.makePreparedStatement(c);

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
				if (c != null) {
						try {
				        c.close();
			      } catch (SQLException e) {
						}
				}
		}
}
```

- deleteAll() 메소드가 클라이언트가 됨
- 전략 오브젝트를 만들고 컨텍스트를 호출하는 책임

```java
public void deleteAll() throws SQLException {
		StatementStrategy st = new DeleteAllStatement(); // 전략 클래스와 오브젝트 생성
		jdbcContextWithStratementStrategy(st); // 전략 오브젝트 전달
}
```

- 마이크로 DI
- DI: 제3자의 도움을 통해 두 오브젝트 사이의 두 오브젝트 사이의 유연한 관계가 설정되도록 만듦
- 의존관계에 있는 두 개의 오브젝트와 오브젝트 팩토리(DI 컨테이너: 관계를 다이나믹하게 설정해줌), 이를 사용하는 클라이언트 즉, 4개의 오브젝트 사이에서 일어남
- 다양한 상황

1. 원시적인 전략 패턴 구조를 따라 클라이언트가 오브젝트 팩토리의 책임을 함께 질 수 있음
2. 클라이언트와 전략(의존 오브젝트)이 결합될 수 있음
3. 클라이언트와 DI 관계에 있는 두 개의 오브젝트가 모두 하나의 클래스 안에 담길 수 있음

- DI가 매우 작은 단위의 코드와 메소드 사이에서 일어나기도 함
- 마이크로 DI: DI의 장점을 단순화해서 IoC 컨테이너의 도움 없이 코드 내에서 적용한 경우
- 코드에 의한 DI라는 의미로 수동 DI라고 부를수도 있음

### 전략 클래스의 추가 정보

- add() 메소드에 적용해보기
- 변하는 부분인 PreparedStatement를 만드는 코드를 AddStatement 클래스로 옮겨 담음

```java
StatementStrategy st = new StatementStrategy() {
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword()); // user라는 부가정보 필요

        return ps;
    }
}
```

```java
public class AddStatement implements StatementStrategy {
		User user;

		public AddStatement(User user) {
				this.user = user;
		}

		public PreparedStatement makePreparedStatement(Connection c) {
        ...
				ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword()); // user라는 부가정보 필요

        ...
    }
}
```

```java
public void add(User user) throws SQLException {
		StatementStrategy st = new AddStatement(user);
		jdbcContextWithStatementStrategy(st);
}
```

### 전략과 클라이언트의 동거

- 현재는 DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 함
  → 기존 UserDao 때보다 클래스 파일 개수가 늘어남
- DAO 메소드에서 부가적인 정보(StatementStrategy에 전달할 User)가 있는 경우, 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수 필요
  → 해당 오브젝트가 사용되는 시점은 컨텍스트가 전략 오브젝트를 호출할 때이므로 잠시 어딘가에 저장해두어야 함

### 로컬 클래스

- StatementStrategy 전략 클래스를 독립된 파일로 만드는 것이 아닌 UserDao 클래스 안에서 내부 클래스로 정의하기 (UserDao의 메소드 로직에 강하게 결합)
- 특정 메소드에서만 사용되는 것이면 **로컬 클래스**로 만들 수 있음
- 장점

  - 클래스 파일이 하나 줄어듦
  - 메소드 안에서 PreparedStatement 생성 로직을 함께 볼 수 있음
  - 로컬 클래스는 자신이 선언된 곳의 정보에 접근할 수 있음

- 중첩 클래스
- 다른 클래스 내부에 정의되는 클래스
  1. 스태틱 클래스(독립적으로 오브젝트로 만들어질 수 있음)
  2. 내부 클래스(자신이 정의된 클래스 오브젝트 안에서만 만들어질 수 있음)
     - 멤버 내부 클래스(오브젝트 레벨에 정의 ex.멤버 필드)
     - 로컬 클래스(메소드 레벨에 정의)
     - 익명 내부 클래스(이름을 갖지 않음)

### 익명 내부 클래스

- 이름을 갖지 않는 클래스
- 클래스 선언과 오브젝트 생성이 결합된 형태
- 상속할 클래스나 구현할 인터페이스를 생성자 대신 사용
- 클래스를 재사용할 필요가 없고 구현한 인터페이스 타입으로만 사용할 경우에 유용
- new 인터페이스이름{} { 클래스 본문 };
- 익명 내부 클래스는 선언과 동시에 오브젝틀를 생성
- 클래스 자신의 타입을 가질 수 없고 구현한 인터페이스 타입의 변수에만 저장 가능

# 4️⃣ 컨텍스트와 DI

## JdbcContext의 분리

- 전략 패턴의 관점에서 UserDao 메소드가 클라이언트이고 익명 내부 클래스로 만들어지는 것이 개별적인 전략임, jdbcContextWithStatementStrategy() 메소드는 컨텍스트임
- 컨텍스트 메소드는 UserDao 내의 PreparedStatement를 실행하는 기능을 가진 메소드에서 공유 가능
- jdbcContextWithStatementStrategy()는 JDBC의 일반적인 작업 흐름을 담고 있으므로 다른 DAO에서도 사용 가능 → 독립 가능

### 클래스 분리

- UserDao의 컨텍스트 메소드를 따로 옮겨둠
- UserDao가 분리된 JdbcContext를 DI 받아서 사용 가능

### 빈 의존관계 변경

- UserDao는 JdbcContext(구체 클래스)에 의존
- jdbcContext 그 자체로 독립적인 JDBC 컨텍스트를 제공해주는 서비스 오브젝트로서 의미가 있으며 구현 방법이 바뀔 가능성이 없음
  → 인터페이스를 구현하지 않음
- 스프링의 빈설정은 런타임 시에 만들어지는 오브젝트 레벨이 의존관계에 따라 정의됨 (클래스 레벨이 아님)
- 의존관계에 따라 XML 수정 필요

## jdbcContext의 특별한 DI

- 현재 구조는 의존 오브젝트의 구현 클래스를 변경할 수 없음

### 스프링 빈으로 DI

- 인터페이스를 사용하지 않았다면 온전한 DI로 볼 수 없음
- 스프링의 DI를 넓게 보았을 때 객체의 생성과 관계설정에 대한 제어권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC의 개념을 포괄 → DI의 기본을 따르고 있다고 볼 수 있음
- jdbcContext를 DI 구조로 만들어야하는 이유

1. jdbcContext가 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이 되기 때문
   - jdbcContext는 JDBC 컨텍스트 메소드를 제공해주는 서비스 오브젝트이므로 싱글톤으로 등록되어 여러 오브젝트에서 공유해 사용하는 것이 이상적
2. ✨ jdbcContext가 DI를 통해 다른 빈에 의존
   - DataSource 오브제트를 주입받도록 되어 있음
   - DI를 위해서는 주입되는 오브젝트와 주입받는 오브젝트 양쪽 모두 빈으로 등록 필요

- 인터페이스가 없다는 것은 UserDao와 JdbcContext가 긴밀한 관계를 가지고 강한 결합을 형성한다는 것 (JPA, 하이버네이트 등을 사용할 경우 완전히 바꾸어야 함, 테스트에서 다른 구현체로 대체해서 사용할 이유가 없음)

### 코드를 이용하는 수동 DI

- 스프링 빈 등록 대신 UserDao 내부에서 직접 DI를 적용하는 방법을 사용할 수 있음
- 싱글톤을 포기해야 함 (DAO마다 하나의 jdbcContext 오브젝트를 갖게 하는 것)
  - JdbcContext에는 내부에 두는 상태정보가 없으므로 메모리에 주는 부담은 거의 없음
  - 자주 만들어졌다가 제거되는 것이 아님 → GC 부담감 적음
  - 제어권은 UserDao에 있음 (전통적인 방법을 따름)
- jdbcContext는 DataSource 타입 빈을 주입받아서 사용해야 함
  - 스프링 빈이 아니어서 DI 컨테이너를 통해 DI 불가능
  - UserDao가 임시로 DI 컨테이너처럼 동작하게 만듦
- DataSource(JdbcContext에 주입할 대상)은 UserDao가 대산 DI 받음

  - 주입 받은 DataSource 빈을 JdbcContext를 만들고 초기화하는 과정에 사용 후 버리면 됨

- 스프링 설정파일에 UserDao와 dataSource 두 개만 빈으로 정의
- setJdbcContext() 제거 후 setDataSource() 메소드 수정

```java
public class UserDao {
    ...
    private JdbcContext jdbcContext;

		// 수정자 메소드 & JdbcContext에 대한 생성, DI 작업 동시 수행
		// DI 컨테이너가 DataSource 오브젝트를 주입해줄 때 호출
    public void setDataSource(DataSource dataSource) {
        this.jdbcContext = new jdbcContext();
        this.jdbcContext.setDataSource(dataSource); // 의존 오브젝트 주입(DI)
        this.setDataSource = dataSource; // 아직 JdbcContext를 적용하지 않은 메소드를 위해 저장
    }
}
```

1. 인터페이스를 사용하지 않는 클래스와의 의존관계이지만 스프링의 DI 사용을 위해 빈으로 등록해서 사용하는 방법
   - 오브젝트 사이의 실제 의존관계가 설정파일에 명확하게 드러남
   - DI의 근본적인 원칙에 부합하지않는 구체적인 클래스와의 관계가 설정에 직접 노출됨
2. 수동으로 DI
   - JdbcContext가 UserDa의 내부에서 만들어지고 사용되면서 관계를 외부에 드러내지 않으
   - 싱글톤으로 만들 수 없고 DI 작업을 위한 부가적인 코드 필요

# 5️⃣ 템플릿과 콜백

- 전략 패턴: 복잡하지만 바뀌지 않는 일정한 패턴을 갖는 작업 흐름이 있고 일부분만 자주 바꿔서 사용해야 하는 경우에 적합
- 템플릿/콜벡 패턴: 전략 패턴의 기본 구조에 익명 내부 클래스 활용
- 템플릿: 전략 패턴의 컨텍스트, 콜백: 익명 내부 클래스로 만들어지는 오브젝트
  - 어떤 목적을 위해 미리 만들어둔 모양이 있는 틀
  - 고정된 작업 흐름을 가진 코드를 재사용함
- 템플릿 메소드 패턴: 고정된 틀의 로직을 가진 템플릿 메소드를 슈퍼클래스에 두고 바뀌는 부분을 서브 클래스의 메소드에 두는 구조
- 콜백: 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트
  - 파라미터로 전달되지만 특정 로직을 담은 메소드의 실행을 위해 사용 (값을 참조)
  - 템플릿 안에서 호출되는 것이 목적
- 핑셔널 오브젝트: 자바에서 메소드 자체를 파라미터로 전달할 수 없어 메소드가 담긴 오브젝트를 전달

## 템플릿/콜백의 동작 원리

### 템플릿/콜백의 특징

- 특정 기능을 위해 한 번 호출되는 경우가 일반적이므로 단일 메소드 인터페이스 사용
- 여러 가지 종류의 전략이 필요하면 하나 이상의 콜백 오브젝트를 사용할 수 있음
- 콜백은 일반적으로 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어짐
- 콜백 인터페이스의 메소드에 있는 파라미터는 템플릿의 작업 흐름 중에 만들어지는 컨텍스트 정보를 전달받을 때 사용

템플릿/콜백의 작업 흐름

- 클라이언트는 템플릿 안에서 실행될 로직을 담은 콜백 객체를 만들고, 콜백이 참조할 정보를 제공함
- 만들어진 콜백은 클라이언트가 템플릿의 메소드를 호출할 떄 파라미터로 전달
- 템플릿은 정해진 작업 흐름을 따라 작업을 진행하다가, 내부에서 생성한 참조정보를 가지고 콜백 객체의 메소드를 호출
- 콜백은 클라이언트 메소드에 있는 정보와 템플릿이 제공한 참조정보를 이용해서 작업을 수행하고 그 결과를 다시 템플릿에 돌려줌
- 템플릿은 콜백이 돌려준 정보를 사용해서 작업을 마저 수행함 최종 결과를 클라이언트에 다시 돌려주는 경우도 있음

- DI 방식의 전략 패턴 구조라고 생각하면 됨
- 클라이언트가 템플릿 메소드를 호출하면서 콜백 객체를 전달 (메소드 레벨에서 일어나는 DI)
- 템플릿이 사용할 콜백 인터페이스를 구현한 객체를 메소드를 통해 주입해주는 DI 작업이 클라이언트가 템플릿의 기능을 호출하는 것과 동시에 일어남

- 매번 메소드 단위로 사용할 객체를 새롭게 전달받음 (일반적인 DI는 템플릿에 인스턴스 변수를 만들어두고 사용할 의존 오브젝트를 수정자 메소드로 받아서 사용)
- 콜백 객체가 내부 클래스로서 자신을 생성한 클라이언트 메소드 내의 정보를 직접 참조한
- 클라이언트와 콜백이 강력하게 결합
- 전략 패턴과 DI의 장점을 익명 내부 클래스 사용 전략과 결합한 독특한 활용 방법
- 하나의 고유한 디자인 패턴으로 기억하는 것이 좋음

### JdbcContext에 적용된 템플릿/콜백

- 조회 작업에서는 보통 템플릿의 작업 결과를 클라이언트에게 리턴
- 템플릿의 작업 흐름이 복잡한 경우 한 번 이상 콜백을 호출하기도 하고 여러 개의 콜백을 클라이언트로 받아서 사용하기도 함

## 편리한 콜백의 재활용

- DAO 메소드에서 매번 익명 내부 클래스를 사용하여 코드를 작성하고 읽기 번거로움

### 콜백의 분리와 재활용

- try/catch/finally를 UserDao 메소드에도 적용
- deleteAll()의 경우 문자열만 바뀌는 것을 확인할 수 있음
- 바뀌지 않는 부분은 executeSql() 메소드로 만들어 변하는 것과 분리함

### 콜백과 템플릿의 결합

- 재사용 가능한 콜백을 담고 있는 메소드를 DAO가 공유할 수 있는 템플릿 클래스 안으로 옮김 (JdbcContext)

- 공통된 목적을 위해 연결되어 동작하는 응집력이 강한 코드는 한 군데 모여있는 것이 유리

## 템플릿/콜백의 응용

- 스프링에는 미리 만들어져 제공되는 수십 가지 템플릿/콜백 클래스와 API가 있음
- 한 애플리케이션 안에서 동시에 여러 종류가 만들어질 수 있다면 템플릿/콜백 패턴 적용 고려
- 고정된 작업 흐름을 갖고 있으면서 자주 반복되는 코드가 있을 경우 중복되는 코드를 분리하기
  - 메소드 분리 → 전략 패턴 적용 → 템플릿/콜백 패턴 적용
- try/catch/finally 블록: 가장 전형적인 템플릿/콜백 패턴의 후보로 일정한 리소스를 만들거나 가져와 작업하면서 예외가 발생할 가능성이 있는 코드임
- 템플릿과 콜백을 찾아낼 때 변하는 코드의 경계를 찾고 그 경계를 사이에 두고 주고 받는 일정한 정보가 있는지 확인

### 테스트와 try/catch/finally

- calcSum(): 파일을 읽거나 처리하는 중 예외 발생 가능성이 있음
- → try/catch/finally 블록 적용

```java
public Integer CalcSum(String filepath) throws IOException {
    BufferedReader br = null;

    try {
        br = new BufferedReader(new FileReader(filepath));
				Integer sum = 0;
        String line = null;
        while((line = br.readLine()) != null) {
            sum += Integer.valueOf(line);
        }
        return sum;
    } catch (IOException e) {
        System.out.println(e.getMessage());
        throw e;
    } finally {
        if(br != null) {
            try { br.close(); }
            catch (IOException e) { System.out.println(e.getMessage()); }
        }
    }
}
```

### 중복의 제거와 템플릿/콜백 설계

- 숫자의 곱을 계산하는 기능 추가
- 파일에 담긴 숫자 데이터를 여러 가지 방식으로 처리하는 기능 추가 필요
- 템플릿/콜백 패턴 적용
- 템플릿과 콜백의 경계를 정하고 템플릿이 콜백에게, 콜백이 템플릿에게 전달하는 내용이 무엇인지 파악하는 것이 중요
- BufferReader를 만들어서 콜백에게 전달, 콜백이 라인을 읽은 후 알아서 처리한 후에 최종 결과만 템플릿에게 돌려줌

```java
public interface BufferedReaderCallback {
		Integer doSomethingWithReader(BufferReader br) throws IOException;
}
```

- 템플릿 부분을 메소드로 분리
- BufferedReaderCallback 인터페이스 타입의 콜백 오브젝트를 받아서 적절한 시점에 실행
- 콜백이 돌려준 결과는 모든 처리를 마친 후 클라이언트에게 돌려줌

```java
public Integer fileReadTemplate(String filepath, BufferedReaderCallback callback) throws IOExeption {
		try {
				...
				int ret = callback.doSomethingWithReader(br);
				...
		}
		...
}
```

- BufferedReader를 이용해 작업을 수행하는 부분은 콜백을 호출하여 처리

```java
public Integer CalcSum(String filepath) throws IOException {
		BufferedReaderCallback sumCallback =
				new BufferedReaderCallback() {
						public Integer doSomethingWithReader(BufferedReader br) throws Exception {
								Integer sum = 0;
					      String line = null;
				        while((line = br.readLine()) != null) {
				            sum += Integer.valueOf(line); // 🚨 중복
				        }
				        return sum;
					}
		};
		return fileReadTemplate(filepath, sumCallback);
}
```

- 곱을 계산하는 콜백을 가진 메소드 추가

```java
public Integer CalcMultiply(String filepath) throws IOException {
		BufferedReaderCallback multiplyCallback =
				new BufferedReaderCallback() {
						public Integer doSomethingWithReader(BufferedReader br) throws Exception {
								Integer multiply = 1;
					      String line = null;
				        while((line = br.readLine()) != null) {
				            multiply *= Integer.valueOf(line); // 🚨 중복
				        }
				        return sum;
						}
				};
		return fileReadTemplate(filepath, multiplyCallback);
}
```

```java
public Integer lineReadTemplate(String filepath, LineCallback callback, int initVal) throws IOException {
    BufferedReader br = null;

    try {
        br = new BufferedReader(new FileReader(filepath));
				Integer res = initVal;
        String line = null;
        while((line = br.readLine()) != null) {
            res = callback.doSomethingWithLine(line, res);
        }
        return res;
    } catch (IOException e) {
				...
    } finally {
				...
    }
}
```

```java
public Integer calcSum(String filepath) throws IOException {
     LineCallback sumCallback =
				new LineCallback {
            public Integer doSomethingWithLine(String line, Integer value) throws IOException {
                return Integer.valueOf(line) + value;
            }
				};
		return lineReadTemplate(filepath, sumCallback, 0);
}

public Integer calcMultiply(String filepath) throws IOException {
    LineCallback multiplyCallback = new LineCallback() {
        public Integer doSomethingWithLine(String line, Integer value) throws IOException {
            return Integer.valueOf(line) * value;
        }
    };
    return lineReadTemplate(filepath, multiplyCallback, 1);
}
```

### 제네릭스를 이용한 콜백 인터페이스

- 파일을 라인 단위로 처리해서 만드는 결과의 타입을 다양하게 가져가고 싶을 때 타입 파라미터라는 개념을 도입한 제네릭스 이용
- 다양한 오브젝트 타입을 지원하는 인터페이스나 메소드 정의 가능

```java
public interface LineCallback<T> {
		T doSomethingWithLine(String line, T value);
}
```

```java
public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, int initVal) throws IOException {
    BufferedReader br = null;

    try {
        br = new BufferedReader(new FileReader(filepath));
				Integer res = initVal;
        String line = null;
        while((line = br.readLine()) != null) {
            res = callback.doSomethingWithLine(line, res);
        }
        return res;
    } catch (IOException e) {
				...
    } finally {
				...
    }
}
```

```java
public String concatenate(String filepath) throws IOException {
    LineCallback<String> concatenateCallback = new LineCallback<String>() {
				public String doSomethingWithLine(String line, String value) throws IOException {
				    return value + line;
        }
		};
    return lineReadTemplate(filepath, concatenateCallback, "");
}
```

# 6️⃣ 스프링의 JdbcTemplate

- 스프링이 제공하는 템플릿/콜백 기술
- JdbcTempate: 스프링이 제공하는 JDBC 코드용 기본 템플릿

```java
public class UserDao {

    private JdbcTemplate jdbcTemplate;

    public void setDataSource(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.dataSource = dataSource;
}
```

## update()

- JdbcTemplate의 콜백: PreparedStatementCreator 인터페이스의 createPreparedStatement() 메소드
- Connection을 제공받아서 PreparedStatement를 만들어 돌려줌
- PreparedStatementCreator 타입의 콜백을 받아서 사용하는 JdbcTemplate의 템플릿 메소드는 update()임

```java
public void deleteAll() {
		this.jdbcTemplate.update(new PreparedStatementCreator() {
        public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
            return con.prepareStatement("delete from users");
        }
    });
}
```

```java
public void deleteAll() {
		this.jdbcTemplate.update("delete from users");
}
```

```java
this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)",
		user.getId(), user.getName(), user.getPassword());
```

## queryForInt()

- 템플릿/콜백 방식을 적용하지 않았던 메소드에 JdbcTemplate 적용
- getCount: SQL 쿼리를 실행하고 ResultSet을 통해 결과값을 가져오는 코드
- ResultSetExtractor 콜백: 템플릿으로 ResultSet을 받고 거기서 추출한 결과를 돌려줌
  - ResultSetExtractor는 제너릭스 타입 파라미터를 갖음
  - ResultSetExtractor 콜백에 지정한 타입은 제네릭 메소드에 적용돼서 query() 템플릿의 리턴 타입도 함께 바뀜
- PreparedStatementCreator 콜백: 템플릿으로 Connection을 받고 PreparedStatement를 돌려줌
  - 해당 콜백에서 리턴하는 값은 결국 템플릿 메소드의 결과로 다시 리턴됨
- jdbcTemplate은 SQL의 실행 결과가 하나의 정수 값이 되는 queryForInt() 메소드 제공

```java
// queryForInt() 메소드는 에석하게 스프링 3.2.2. 이후로는 Deprecated 됨
public int getCount() throws SQLException {
    return this.jdbcTemplate.queryForObject("select count(*) from users", Integer.class);
}
```

## queryForObject

- ResultSet의 결과를 Uesr 오브젝트를 만들어 프로퍼티에 넣어주어야 함
- ResultSetExtractor대신 RowMappper 콜백 사용
  - 공통점: 템플릿으로부터 ResultSet을 받아 필요한 정보를 추출한다
  - 다른점: ResultSetExtractor는 한 번 전달받아 알아서 추출 작업 진행, RowMapper는 ResultSet 로우 하나를 매핑하기 위해 사용하므로 row수 만큼 여러번 호출됨
- 첫번째 파라미터: PreparedStatement를 만들기 위한 SQL
- 두번째 파라미터: 바인딩할 값(가변인자 대신 Object 타입 배열 사용)
- RowMapper가 리턴한 User 오브젝트는 queryForObject() 메소드의 리턴 값으로 get() 메소드에 전달
- queryForObject()는 SQL 실행 후 로우의 개수가 하나가 아니라면 예외를 던짐(EmptyResultDataAccessException)

## query()

### 기능 정의와 테스트 작성

```java
@Test
public void getAll() {
		dao.deleteAll();

    List<User> users0 = dao.getAll();
    assertThat(users0.size(), is(0));

    dao.add(user1);
    List<User> users1 = dao.getAll();
    assertThat(users1.size(), is(1));
    checkSameUser(user1, users1.get(0));

		...
}

private void checkSameUser(User user1, User user2) {
    assertThat(user1.getId(), is(user2.getId()));
    assertThat(user1.getName(), is(user2.getName()));
    assertThat(user1.getPassword(), is(user2.getPassword()));
}
```

### query() 템플릿을 이용하는 getAll() 구현

- 작성한 테스트를 성공시키는 getAll() 구현

```java
public List<User> getAll() {
		return this.jdbcTemplate.query("select * from users order by id",
				new RowMapper<User>() {
						public User mapRow(ResultSet rs, int rowNum) throws SQLException {
					      User user = new User();
		            user.setId(rs.getString("id"));
		            user.setName(rs.getString("name"));
		            user.setPassword(rs.getString("password"));
		            return user;
        }
		});
}
```

### 테스트 보완

- 네거티브 테스트의 중요성

```java
public void getAll() {
		dao.deleteAll();

		List<User> users0 = dao.getAll();
		assertThat(users0.size(), is(0));
}
```

## 재사용 가능한 콜백의 분리

### DI를 위한 코드 정리

- 불필요한 DataSource 변수를 제거한 UserDao의 DI 코드

```java
public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
}
```

### 중복 제거

- get()과 getAll()의 RowMapper 내용 중복 → 따로 분리

```java
public class UserDao {
		private RowMapper<User> userMapper =
				new RowMapper<User>() {
						public User mapRow(ResultSet rs, int rowNum) throws SQLException {
					      User user = new User();
		            user.setId(rs.getString("id"));
		            user.setName(rs.getString("name"));
		            user.setPassword(rs.getString("password"));
		            return user;
        }
		};
}
```

### 템플릿/콜백 패턴과 UserDao

- UserDao; User 정보를 DB에 넣거나 가져오거나 조작하는 방법에 대한 핵심적인 로직이 담김
  - 사용할 테이블과 필드정보의 변경은 UserDao의 거의 모든 코드를 바꿈 → 응집도가 높음
- JdbcTemplate: JDBC API를 사용하는 방식, 예외처리, 리소스의 반납, DB연결을 어떻게 가져올지에 관한 책임과 관심
  - 변경이 일어나도 UserDao 코드에 영향을 주지 않음 → 낮은 결합도
- UserDao와 JdbcTemplate 사이에는 강한 결합을 가지고 있음
- 개선 가능한 부분
  1. userMapper가 인스턴스 변수로 설정되어 있고, 한 번 만들어지면 변경 불가능
     - UserMapper를 독립된 빈으로 만들고 DI 하게 만들기
  2. DAO 내 사용중인 SQL 문장을 코드가 아니라 외부 리소스에 담고 이를 읽어와 사용하게 하기

### 정리

- try/catch/finally 블록: JDBC와 같은 예외가 발생할 가능성이 있으며 공유 리소스의 반환이 필요한 코드
- 전략 패턴: 일정한 작업 흐름이 반복되면서 그중 일부 기능만 바뀌는 코드가 존재
  - 컨텍스트: 바뀌지 않는 부분
  - 전략: 바뀌는 부분
  - 인터페이스 사용
- 같은 애플리케이션 안에서 여러 가지 종류의 전략을 다이내믹하게 구성하고 사용해야 한다면 컨텍스트를 이용하는 클라이언트 메소드에서 직접 전략을 정의하고 제공하게 만들기
- 클라이언트 메소드 안에 익명 내부 클래스를 사용해서 전략 오브젝트를 구현하면 코드도 간결해지고 메소드의 정보를 직접 사용할 수 있어 편리함
- 컨텍스트가 하나 이상의 클라이언트 오브젝트에 사용 → 클래스를 분리해서 공유하도록 만들기
- 컨텍스트는 별도의 빈으로 등록해서 DI 받거나 클라이언트 클래스에서 직접 생성해서 사용
  - 클래스 내부에서 컨텍스트를 사용할 때 컨텍스트가 의존하는 외부의 오브젝트가 있다면 코드를 이용해서 직접 DI 해줄 수 있음
- 템플릿/콜백 패턴: 단일 전략 메소드를 갖는 전략 패턴, 익명 내부 클래스 사용
  - 매번 전략을 새로 만들어 사용, 컨텍스트 호출과 동시에 전략 DI를 수행
- 콜백의 코드에도 일정한 패턴이 반복된다면 콜백을 템플릿에 넣고 재활용하는 것이 편리함
- 템플릿과 콜백의 타입이 다양하게 바뀔 수 있다면 제네릭스 이용하기
- 스프링은 JDBC 코드 작성을 위해 JdbcTemplate을 기반으로 하는 다양한 템플릿과 콜백을 제공
- 템플릿은 한 번에 하나 이상의 콜백을 사용할 수도 있고, 하나의 콜백을 여러 번 호출할 수 있음
- 템플릿/콜백을 설계할 때는 템플릿과 콜백 사이에 주고받는 정보에 관심을 두어야 함
